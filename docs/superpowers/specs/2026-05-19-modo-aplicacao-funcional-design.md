# Modo Aplicação Funcional — Design

**Data:** 2026-05-19
**Projeto:** pitch-vilelaai-metropole-parque
**Arquivo afetado:** [index.html](../../../index.html)
**Status:** Design aprovado, aguardando revisão do spec antes do plano de implementação

---

## Contexto

O `index.html` é um pitch deck single-file (1920×1080, GitHub Pages) para a submissão **VilelaAI / Metrópole Parque IMD 2026**. Contém:

- **Modo Slides** (default) — 9 slides, navegação por teclado, modo gravação.
- **Modo Aplicação** (tecla `A`) — overlay que esconde os slides e mostra um protótipo do **KairOS Studio** com 8 telas via sidebar (Overview, Domain Catalog, Agents, Assertions, Playground, Observability, FinOps, Compliance).

Hoje o Modo Aplicação tem partes interativas (catálogo → playground, pipeline animado, KPIs por período em Observability) mas a maioria das telas é mockup estático e não há estado compartilhado entre telas.

## Objetivo

Deixar o Modo Aplicação **o mais funcional possível**, com dois requisitos simultâneos:

1. **Gravação de vídeo**: o roteiro de cliques pré-definido roda fluido na câmera, sem surpresa.
2. **Exploração pública pelo júri**: cliques aleatórios não quebram a ilusão; tudo que parece clicável responde.

## Decisões fundamentais

| # | Decisão |
|---|---|
| D1 | **Roteiro de demo** (6 telas): Overview → Catálogo → Playground → Observability → FinOps → Compliance |
| D2 | **Modelo de estado**: compartilhado por domínio — selecionar 1 domínio propaga em todas as telas |
| D3 | **Catálogo**: 3 GovAI "featured" (cross-screen rico) + 6 cards placeholder com badge "Em onboarding · ETA" |
| D4 | **Tríade featured**: **ContrIA** (governo-contratos · Lei 14.133/21), **LicitIA** (licitações · Lei 14.133/21), **PrivacIA** (lgpd) |
| D5 | **Origem dos dados**: embedados como JSON inline a partir do repo privado `VilelaAI/kairos-dominios-pro` (assertions, cenários, agentes reais) |
| D6 | **Profundidade de interação**: drill-down completo (painel lateral) + simulação de atividade ao vivo (ticker, novas traces, alertas) |
| D7 | **Agents e Assertions** (fora do roteiro): funcionais com dados das 3 GovAI; filtros reais; drill-down |
| D8 | **Arquitetura JS**: store reativo vanilla (`state` + `subscribe`), sem CDN, offline-safe |

## Arquitetura

### 1. Store reativo central

No topo do `<script>` existente em `index.html`, antes da lógica de slides:

```js
const state = {
  currentDomain: null,        // 'contria' | 'licitia' | 'privacia' | null  (apenas featured viram domínio ativo)
  currentScreen: 'overview',  // espelha o sidebar do KairOS Studio
  period: '7d',               // '1d' | '7d' | '30d'  (sync Observability, FinOps, Overview)
  drillDown: null,            // { type: 'agent'|'assertion'|'trace'|'compliance-item'|'placeholder'|'model'|'budget', payload }
  ticker: {
    recentTraces: [...],       // últimas 10 execuções para Overview feed + Observability tail
    lastTick: Date.now(),
    paused: false,
  },
  filters: {
    catalog: { regulator: 'all', query: '' },
    agents:  { role: 'all' },
    assertions: { status: 'all' },
    compliance: { category: 'lei14133' },
  },
};

const subscribers = new Set();
function setState(patch) {
  Object.assign(state, patch);
  subscribers.forEach(fn => fn(state));
}
function subscribe(fn) { subscribers.add(fn); return () => subscribers.delete(fn); }
```

Padrão de uso por tela: cada tela tem um `renderXxx(state)` registrado via `subscribe(renderXxx)` uma única vez no setup. O render lê apenas os pedaços de `state` que importam e atualiza só os nós DOM da sua tela.

### 2. Dados embedados

Bloco `const KAIROS_DATA = {...}` no `<script>` (logo abaixo do store), serializado a partir dos arquivos reais do repo `kairos-dominios-pro`:

```js
const KAIROS_DATA = {
  featured: {
    contria: {
      id: 'contria',
      name: 'ContrIA',
      fullName: 'Contratos Administrativos',
      regulators: ['Lei 14.133/21', 'TCU', 'CGU'],
      agents: [                          // de squad-negocial.yaml curado (4-6 agentes)
        { id, role, specialty, description }
      ],
      assertions: [                      // de assertions.md curado (8-12 assertions)
        { name, description, category }
      ],
      scenarios: [                       // de cenarios.jsonl curado (3 cenários por domínio)
        { id, agent, input, expected_keys, output_mock }
      ],
      metrics7d: { execs: 4521, tokens: '15.2M', cost: 'R$ 384', approval: 99.1 },
      metrics1d: { ... }, metrics30d: { ... },
      finops: { models: [{name, cost, tokens, pct}, ...], topPrompts: [...] },
      compliance: { items: [{label, regulator, lastCheck, status}, ...] },
    },
    licitia: { /* mesma estrutura */ },
    privacia: { /* mesma estrutura */ },
  },
  placeholders: [
    { id: 'transparencia', name: 'TransparêncIA', regulators: ['LAI','LRF'], eta: '60d', cluster: 'govai' },
    { id: 'ouvidoria',     name: 'OuvidorIA',     regulators: ['Lei 13.460/17'], eta: '90d', cluster: 'govai' },
    { id: 'servidor',      name: 'ServidorIA',    regulators: ['eSocial gov'], eta: '90d', cluster: 'govai' },
    { id: 'saude',         name: 'Saúde',         regulators: ['CFM','ANS'], eta: '120d', cluster: 'setorial' },
    { id: 'educacao',      name: 'Educação Inclusiva', regulators: ['LBI','BNCC'], eta: '120d', cluster: 'setorial' },
    { id: 'tributario',    name: 'Tributário',    regulators: ['CTN','RFB'], eta: 'roadmap', cluster: 'setorial' },
  ],
  global: {
    execs7d: 12847, tokens7d: '43.9M', cost7d: 'R$ 891', approval7d: 98.7,
    execs1d: 1835, /* ... */
    execs30d: 51284, /* ... */
  },
};
```

A serialização é feita **uma vez** antes da implementação, via `gh api ... | base64 -d` para cada arquivo dos 3 domínios, seguida de curadoria manual.

### 3. Drill panel único

Um único elemento `<aside id="drillPanel">` em `#kairosApp` (slide-in da direita, ~480px de largura, max-height 92vh, scroll interno). O renderer `renderDrillPanel(state)` lê `state.drillDown` e troca o conteúdo conforme o `type`. Fecha em:

- Tecla `Esc`
- Clique no backdrop
- Tecla `D` (toggle)

Sem foco trap (é prototipo).

### 4. Ticker de atividade

Único `setInterval(tick, 9000)` controla:

- **Incremento de execs7d**: `+random(6,18)` por tick, animado no número via `animateNumber()`.
- **Nova trace**: gera 1 entrada (rotação ponderada featured 70% / placeholder 30%) e `unshift()` em `state.ticker.recentTraces`, mantém último 10. Aparece no topo do trace table com classe CSS `.trace-new` (fade-in, depois remove a classe).
- **Pulse no budget alert**: animação CSS pura `@keyframes pulse-budget` — não precisa JS.

**Pausa automática**:
- Quando `state.drillDown !== null` (drill aberto)
- Quando `state.currentScreen === 'playground'` (foco no ato)
- Quando `document.body.classList.contains('recording-mode')` (modo gravação)

**Failsafe**: tecla `T` toggle manual via `state.ticker.paused`.

**Importante**: a pausa do ticker afeta **apenas a injeção automática de novas traces e o incremento de KPI**. Quando o usuário aperta "Executar" no Playground, a trace gerada é registrada em `state.ticker.recentTraces` independente do estado de pausa — isso é um efeito de ação humana, não simulação.

### 5. Atalhos de teclado (Modo Aplicação)

Estendendo o handler existente em `index.html` (linha ~3948), quando `document.body.classList.contains('app-mode')`:

| Tecla | Ação | Condição |
|-------|------|----------|
| `Esc` | Se drill aberto: fecha drill. Senão: sai do Modo Aplicação (volta pros slides). | sempre |
| `D`   | Toggle do drill panel (fecha se aberto; sem efeito se fechado). | `app-mode` ativo |
| `T`   | Toggle `state.ticker.paused`. Mostra tooltip "Ticker pausado/retomado" por 1.5s. | `app-mode` ativo |

As setas/Space/PageUp/Down/Home/End/0-8 continuam **bloqueadas** em `app-mode` (não navegam slides). Teclas `R` (gravação) e `F` (fullscreen) continuam ativas em ambos os modos.

## Por tela: contrato funcional

### Overview

- Header: tenant `vilelaai-prod`, period tabs (1d/7d/30d) que setam `state.period`.
- Hero KPIs (4): execuções totais (ticker animado), tokens, custo, aprovação. Leem `KAIROS_DATA.featured[currentDomain].metricsXd` quando `currentDomain` setado, senão `KAIROS_DATA.global`.
- 3 quick-cards (ContrIA/LicitIA/PrivacIA): clique → `setState({ currentDomain: id, currentScreen: 'playground' })`.
- Activity feed: top 5 de `state.ticker.recentTraces`. Cada linha clicável → `setState({ drillDown: { type: 'trace', payload: trace } })`.
- Banner "Vendo: {nome}" com X quando `currentDomain` setado; X limpa o domínio.

### Domain Catalog

- Grid 3 cols × 3 rows. 3 featured (glow + métricas reais) + 6 placeholder (badge "Em onboarding · ETA {eta}", visual mais sóbrio).
- Filter chips: `Todos · Procurement (Lei 14.133) · LGPD · GovAI · Em onboarding`. Setam `state.filters.catalog.regulator`.
- Search input: live filter por nome/regulador (setam `state.filters.catalog.query`).
- Clique em featured → `setState({ drillDown: { type: 'domain', payload: domain } })`. Drill mostra: descrição, lista de agentes, top 5 assertions, 3 scenarios, botão "Executar no Playground" que faz `setState({ currentDomain: id, currentScreen: 'playground', drillDown: null })`.
- Clique em placeholder → `setState({ drillDown: { type: 'placeholder', payload: domain } })`. Drill mostra: roadmap, "Em onboarding · ETA {eta}", contato `contato@vilela.ai`.

### Playground

- Cabeçalho: "Pipeline anti-alucinação · {domain.fullName}" (lê `currentDomain`; se null, mantém comportamento atual "selecione um domínio").
- Scenario picker (`<select>`): scenarios do `KAIROS_DATA.featured[currentDomain].scenarios`. Trocar scenario reseta o pipeline.
- Pipeline 4 layers — nomes das assertions vêm das **top 4 assertions** do domínio, escolhidas no `KAIROS_DATA` (não hardcoded "M.S.A.").
- Output mock: lê `scenario.output_mock` do dado embedado.
- "Executar" → roda animação (já existe) + ao final faz `state.ticker.recentTraces.unshift({...})` para a trace aparecer em Observability/Overview.
- "Ver na Observabilidade" preserva `currentDomain`.

### Observability

- Period tabs já existem — agora setam `state.period` (sync com Overview/FinOps).
- KPIs: lê `KAIROS_DATA.featured[currentDomain].metricsXd` quando `currentDomain` setado, senão `KAIROS_DATA.global`.
- Trace table: 10 linhas vindas de `state.ticker.recentTraces` (filtradas por `currentDomain` se setado).
- Cada linha clicável → `setState({ drillDown: { type: 'trace', payload: trace } })`. Drill mostra os **4 steps do pipeline** com input, output, assertions checadas, latência por step.
- Live tail: novas traces entram com classe `.trace-new` (animação fade-in 600ms).

### FinOps

- Period tabs sync `state.period`.
- Cost-by-model card: lê `KAIROS_DATA.featured[currentDomain].finops.models` quando `currentDomain` setado, senão um agregado em `KAIROS_DATA.global.finops.models`.
- Budget alert card: pulsing (CSS), texto reativo ao `currentDomain` ("ContrIA · 78% do orçamento" / "Sistema · 64% do orçamento agregado").
- Clique em modelo → drill: top-5 prompts por custo (`finops.topPrompts` filtrado pelo modelo).
- Clique no alerta → drill: histórico de consumo + projeção EOM.

### Compliance

- 4 categorias clicáveis: Lei 14.133/21, LGPD, LAI, Acessibilidade. Setam `state.filters.compliance.category`.
- Quando categoria ativada → mostra cards de regulamentações cobertas, com pass/fail da última checagem (vem do `KAIROS_DATA.featured[currentDomain].compliance.items` ou agregado global).
- Clique em item → drill: definição da assertion, scenarios que testam, taxa de aprovação 7d.
- Quando `currentDomain` setado, categorias do domínio recebem badge "Aplicável".

### Agents (não-roteiro)

- Lista de agentes das 3 GovAI featured. Quando `currentDomain` setado, mostra só os do domínio.
- Filter chips por papel/specialty — setam `state.filters.agents.role`, filtragem real no render.
- Clique em row → drill: papel completo, assertions vinculadas (links pra Assertions), scenarios em que aparece.

### Assertions (não-roteiro)

- Lista de assertions das 3 GovAI. Quando `currentDomain` setado, mostra só as do domínio.
- Filter chips: Todos · Passing · Warning · Failing — filtragem real.
- Clique em row → drill: definição, scenarios que testam, taxa de aprovação histórica (mock).

## Etapa preparatória: extração e curadoria dos dados reais

**Antes** da implementação:

1. Fetch via `gh api repos/VilelaAI/kairos-dominios-pro/contents/dominios/{slug}/{file} --jq '.content' | base64 -d` para cada `{slug}` em `[governo-contratos, licitacoes, lgpd]` e cada `{file}` em `[DOMINIO.md, squad-negocial.yaml, guardrails.yaml, assertions/assertions.md, cenarios/cenarios.jsonl]`.
2. Salvar temporariamente em `tmp/dominios-extract/` (gitignored).
3. **Curadoria** (manual ou script):
   - 4-6 agentes por domínio (descartar duplicatas, manter os mais centrais)
   - 8-12 assertions por domínio (priorizar nomes "demonstráveis" e citações legais fortes)
   - 3 cenários por domínio (input ≤ 200 chars, expected curto, output_mock plausível e impactante)
4. Serializar em JS literal e colar como `KAIROS_DATA` no `index.html`.
5. Apagar `tmp/dominios-extract/`.

## Plano de fases (alto nível, detalhamento no plano de implementação)

1. **Fase 0 — Higiene**: lidar com o diff pendente de `index.html` (+4012/−865) antes de começar. Recomendação: commitar como está, partir desse ponto.
2. **Fase 1 — Extração e curadoria de dados** (~30min): script ou manual com `gh api`, curadoria, gera o objeto `KAIROS_DATA`.
3. **Fase 2 — Store reativo** (~45min): instalar `state`/`subscribe`/`setState` no topo do `<script>`. Adaptar lógica existente de sidebar/playground/observability pra usar o store sem regredir comportamento.
4. **Fase 3 — Drill panel** (~1h): HTML do `<aside>`, CSS slide-in, renderer com switch por `type`.
5. **Fase 4 — Tela Catalog v2** (~1h): refatorar para usar `KAIROS_DATA.featured` + `placeholders`, chips funcionais, search, click → drill.
6. **Fase 5 — Tela Playground v2** (~1h): scenario picker, assertions por domínio, output do dado real, registro de trace ao executar.
7. **Fase 6 — Tela Observability v2** (~45min): period sync, trace table conectada ao ticker, drill por trace.
8. **Fase 7 — Tela FinOps v2** (~1h): cost-by-model reativo, budget alert pulsante, drills.
9. **Fase 8 — Tela Compliance v2** (~45min): categorias funcionais, drill por item.
10. **Fase 9 — Telas Agents/Assertions v2** (~1h): listas, chips, drills.
11. **Fase 10 — Tela Overview v2** (~45min): hero KPIs reativos, activity feed conectado ao ticker, quick-cards.
12. **Fase 11 — Ticker** (~30min): `setInterval`, pausa automática, tecla `T`.
13. **Fase 12 — QA roteiro** (~30min): rodar o roteiro de 6 telas no Chrome em viewport 1920×1080, com modo gravação ativo, validar zero distrações.

**Tempo total estimado**: ~10h de trabalho focado. Pode ser quebrado em 2-3 sessões.

## Riscos e mitigações

| Risco | Mitigação |
|-------|-----------|
| Ticker distrai na gravação | Auto-pausa em `recording-mode`; tecla `T` failsafe global |
| Drill panel não fecha rápido o suficiente na demo | 3 caminhos de fechamento (Esc, clique fora, `D` toggle) |
| Dados reais YAML/JSONL grandes demais | Curadoria reduz a 4-6 agentes, 8-12 assertions, 3 cenários por domínio |
| Inconsistência: card "Saúde" placeholder colide com identidade visual featured | Placeholders têm visual deliberadamente mais sóbrio (sem glow, com badge cinza) |
| Refactor JS quebra slides | Render functions isoladas no escopo `#kairosApp`; lógica de slides intacta |
| Diff pendente conflita | Commitar o diff atual antes de começar (fase 0) |
| Repo `kairos-dominios-pro` privado bloqueia jurado | Dados são **embedados** no HTML, não fetched — jurado nunca tenta acessar o repo |
| Performance: subscribe em todos os screens dispara render demais | Cada render é idempotente e barato; nada de virtual DOM, só `textContent`/`classList` em nós existentes |

## Critérios de aceitação

1. Apertando `A`, entrando no Modo Aplicação, o usuário consegue:
   - Ver Overview com hero KPIs, ticker rodando, 3 quick-cards GovAI
   - Clicar quick-card ContrIA → ir pro Playground com domínio setado, executar, ver trace aparecer em Observability
   - Voltar ao Overview e ver a trace nova no activity feed
   - Ir pra FinOps e ver custos do domínio ContrIA, clicar num modelo, ver drill com top-5 prompts
   - Ir pra Compliance, clicar em Lei 14.133, ver assertions com pass/fail
   - Pressionar Esc em qualquer drill panel fecha
2. Pressionando `R` → `recording-mode` ativa → ticker pausa → nav-controls somem.
3. Clicar em qualquer placeholder do catálogo abre drill elegante "Em onboarding · ETA".
4. Filtros de chips e search no Catalog filtram cards de verdade.
5. Telas Agents e Assertions, mesmo fora do roteiro, têm listas, filtros funcionais e drills.
6. Sem nenhum erro no console.
7. Modo Slides (sair com Esc) continua 100% funcional, sem regressão.

## Fora de escopo

- Conectar a um backend real (mock everything inline).
- Persistência (sem `localStorage` — recarregar limpa estado).
- Multi-tenant real (tenant é label estático).
- Internacionalização (pt-BR somente).
- Mobile/responsive abaixo de 1920×1080 (auto-scale já existe e basta).
- Acessibilidade ARIA completa (é protótipo de pitch, não produto).
- Animação fancy nos slides (somente Modo Aplicação).

## Referências

- Repo de domínios: `VilelaAI/kairos-dominios-pro` (privado)
- Arquivos por domínio: `DOMINIO.md`, `squad-negocial.yaml`, `guardrails.yaml`, `assertions/assertions.md`, `cenarios/cenarios.jsonl`
- Domínios usados: `governo-contratos` (ContrIA), `licitacoes` (LicitIA), `lgpd` (PrivacIA)
