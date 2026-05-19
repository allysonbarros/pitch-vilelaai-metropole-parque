# Modo Aplicação Funcional Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Transformar o Modo Aplicação do pitch (overlay KairOS Studio em `index.html`) em um protótipo funcional com estado compartilhado por domínio, drill-down em todas as 8 telas, dados reais embedados (ContrIA + LicitIA + PrivacIA) e simulação de atividade ao vivo — preparando o deck para gravação de vídeo e exploração livre pelo júri da Metrópole Parque IMD 2026.

**Architecture:** Store reativo vanilla (`state` + `subscribe` + `setState`) injetado no `<script>` existente. Dados curados do repo privado `VilelaAI/kairos-dominios-pro` embedados como objeto literal `KAIROS_DATA`. Cada tela tem um renderer que assina o store e atualiza só seus nós DOM. Um único `<aside id="drillPanel">` slide-in renderiza detalhes via switch por `type`. Ticker `setInterval` simula execuções com auto-pausa em recording-mode / Playground / drill aberto.

**Tech Stack:** HTML5 + CSS3 + JavaScript ES2020 vanilla (sem deps, sem CDN). GitHub Pages para deploy. `gh api` (CLI do GitHub) para extração one-time de dados do repo de domínios.

**Spec:** [docs/superpowers/specs/2026-05-19-modo-aplicacao-funcional-design.md](../specs/2026-05-19-modo-aplicacao-funcional-design.md) (commit `8aab5c2`)

---

## Nota de Segurança sobre Inner HTML

Várias tasks deste plano usam atribuição direta a `.innerHTML` para renderizar templates. **Isto é seguro neste contexto** porque:

1. Toda string interpolada vem ou de `KAIROS_DATA` (dados curados estaticamente no build, sob nosso controle) ou de objetos construídos por nós dentro do próprio `<script>` (traces sintéticos, métricas calculadas).
2. O único campo de input livre do usuário é `state.filters.catalog.query` — e ele **não é renderizado em HTML**, é usado apenas em `.includes()` no filtro JS.
3. Não há fetch, sem CORS, sem dados externos em runtime. Não há autenticação nem dados de outros usuários. É uma demo single-file estática deployada no GitHub Pages.

Se em algum momento o protótipo evoluir para aceitar input do usuário em campos exibidos (ex: comentários, anotações), substitua os `innerHTML` por `textContent` + `createElement` ou adote DOMPurify.

---

## File Structure

Único arquivo de produção é `index.html` — todo o trabalho acontece dentro dele. Arquivos auxiliares:

| Caminho | Propósito | Persistência |
|---|---|---|
| `index.html` | Modificações em 4 regiões: bloco `<style>`, HTML do `#kairosApp`, bloco `<script>`, e novo bloco `KAIROS_DATA` | Persistente |
| `tmp/dominios-extract/` | Dump bruto dos arquivos do `kairos-dominios-pro` durante curadoria | Gitignored, removido ao final |
| `.gitignore` | Adicionar `tmp/` | Persistente |
| `docs/superpowers/specs/2026-05-19-modo-aplicacao-funcional-design.md` | Spec aprovado | Persistente |
| `docs/superpowers/plans/2026-05-19-modo-aplicacao-funcional.md` | Este plano | Persistente |

**Sem novos arquivos JS/CSS.** Spec define inline para offline-safety na gravação.

---

## Verification Approach

Este projeto **não tem test runner** (é um pitch deck single-file). TDD tradicional não se aplica — adaptamos a disciplina para **verificação manual em browser + DevTools**, antes e depois de cada mudança:

- **Antes**: abrir `index.html` em Chrome, pressionar `A` para entrar no Modo Aplicação, reproduzir o estado atual (ou ausência da feature), confirmar mudança necessária.
- **Depois**: recarregar (Cmd+R) com DevTools abertos (F12), executar a sequência da verificação, confirmar comportamento esperado, **conferir Console limpo** (zero erros JS).
- **Regressão dos Slides**: após cada mudança não-trivial, sair (`Esc`), navegar slides 1→9 com setas, voltar ao Modo Aplicação. Slides nunca podem regredir.

**Servir local** durante dev: `python3 -m http.server 8000`, abrir `http://localhost:8000`. Para o restante deste plano, assumo que o leitor mantém um servidor local rodando.

---

## Phase 0 — Higiene

### Task 0.1: Commit do diff pendente em `index.html`

**Files:**
- Modify: `index.html` (apenas o commit; não toca conteúdo)

- [ ] **Step 1: Inspecionar o diff pendente**

```bash
git status
git diff --stat index.html
```

Expected: `index.html` modificado, diff ~+4012/−865.

- [ ] **Step 2: Skim do diff**

```bash
git diff index.html | head -200
```

Confirme que são mudanças coerentes (telas adicionadas, estilo refinado). Se algo parecer fora do escopo, pare e ajuste.

- [ ] **Step 3: Stage e commit**

```bash
git add index.html
git commit -m "WIP: estado base do Modo Aplicacao antes de refatorar

Captura o estado atual do KairOS Studio overlay com 8 telas
mockup antes da refatoracao para estado compartilhado por
dominio + drill-down + ticker.

Co-Authored-By: Claude Opus 4.7 (1M context) <noreply@anthropic.com>"
```

- [ ] **Step 4: Verificar working tree limpo**

```bash
git status
```

Expected: `nothing to commit` (exceto arquivos novos em `docs/superpowers/plans/`).

---

## Phase 1 — Extração e Curadoria de Dados

### Task 1.1: Setup do diretório de extração e gitignore

**Files:**
- Create: `.gitignore`, `tmp/dominios-extract/`

- [ ] **Step 1: Criar `.gitignore`**

Crie `.gitignore` na raiz com conteúdo:

```
# Diretorio temporario de extracao (Modo Aplicacao)
tmp/

# OS / editor
.DS_Store
*.swp
```

- [ ] **Step 2: Criar diretório**

```bash
mkdir -p tmp/dominios-extract/contria tmp/dominios-extract/licitia tmp/dominios-extract/privacia
```

- [ ] **Step 3: Commit do gitignore**

```bash
git add .gitignore
git commit -m "Add .gitignore (tmp/, .DS_Store)

Co-Authored-By: Claude Opus 4.7 (1M context) <noreply@anthropic.com>"
```

### Task 1.2: Extrair arquivos do ContrIA (`governo-contratos`)

**Files:**
- Create: `tmp/dominios-extract/contria/DOMINIO.md`, `squad-negocial.yaml`, `guardrails.yaml`, `assertions.md`, `cenarios.jsonl`

- [ ] **Step 1: Verificar acesso ao repo**

```bash
gh repo view VilelaAI/kairos-dominios-pro
```

Expected: metadata do repo (privado). Se 404, rode `gh auth status` e garanta acesso. Sem acesso → abortar e escalar ao usuário.

- [ ] **Step 2: Extrair os 5 arquivos**

```bash
for FILE in DOMINIO.md squad-negocial.yaml guardrails.yaml; do
  gh api "repos/VilelaAI/kairos-dominios-pro/contents/dominios/governo-contratos/${FILE}" --jq '.content' | base64 -d > "tmp/dominios-extract/contria/${FILE}"
done

gh api "repos/VilelaAI/kairos-dominios-pro/contents/dominios/governo-contratos/assertions/assertions.md" --jq '.content' | base64 -d > tmp/dominios-extract/contria/assertions.md
gh api "repos/VilelaAI/kairos-dominios-pro/contents/dominios/governo-contratos/cenarios/cenarios.jsonl" --jq '.content' | base64 -d > tmp/dominios-extract/contria/cenarios.jsonl
```

- [ ] **Step 3: Verificar arquivos com conteúdo**

```bash
wc -l tmp/dominios-extract/contria/*
```

Expected: cada arquivo > 0 linhas. Se vazio, verificar caminho real com `gh api repos/VilelaAI/kairos-dominios-pro/contents/dominios/governo-contratos`.

### Task 1.3: Extrair arquivos do LicitIA (`licitacoes`)

**Files:**
- Create: `tmp/dominios-extract/licitia/{DOMINIO.md,squad-negocial.yaml,guardrails.yaml,assertions.md,cenarios.jsonl}`

- [ ] **Step 1: Extrair**

```bash
for FILE in DOMINIO.md squad-negocial.yaml guardrails.yaml; do
  gh api "repos/VilelaAI/kairos-dominios-pro/contents/dominios/licitacoes/${FILE}" --jq '.content' | base64 -d > "tmp/dominios-extract/licitia/${FILE}"
done
gh api "repos/VilelaAI/kairos-dominios-pro/contents/dominios/licitacoes/assertions/assertions.md" --jq '.content' | base64 -d > tmp/dominios-extract/licitia/assertions.md
gh api "repos/VilelaAI/kairos-dominios-pro/contents/dominios/licitacoes/cenarios/cenarios.jsonl" --jq '.content' | base64 -d > tmp/dominios-extract/licitia/cenarios.jsonl
```

- [ ] **Step 2: Verificar**

```bash
wc -l tmp/dominios-extract/licitia/*
```

Expected: 5 arquivos não-vazios.

### Task 1.4: Extrair arquivos do PrivacIA (`lgpd`)

**Files:**
- Create: `tmp/dominios-extract/privacia/{DOMINIO.md,squad-negocial.yaml,guardrails.yaml,assertions.md,cenarios.jsonl}`

- [ ] **Step 1: Extrair**

```bash
for FILE in DOMINIO.md squad-negocial.yaml guardrails.yaml; do
  gh api "repos/VilelaAI/kairos-dominios-pro/contents/dominios/lgpd/${FILE}" --jq '.content' | base64 -d > "tmp/dominios-extract/privacia/${FILE}"
done
gh api "repos/VilelaAI/kairos-dominios-pro/contents/dominios/lgpd/assertions/assertions.md" --jq '.content' | base64 -d > tmp/dominios-extract/privacia/assertions.md
gh api "repos/VilelaAI/kairos-dominios-pro/contents/dominios/lgpd/cenarios/cenarios.jsonl" --jq '.content' | base64 -d > tmp/dominios-extract/privacia/cenarios.jsonl
```

- [ ] **Step 2: Verificar**

```bash
wc -l tmp/dominios-extract/privacia/*
```

### Task 1.5: Curar dados de ContrIA — Agents

**Files:**
- Read: `tmp/dominios-extract/contria/squad-negocial.yaml`
- Create: `tmp/dominios-extract/contria/_curado.md`

- [ ] **Step 1: Ler o YAML**

```bash
cat tmp/dominios-extract/contria/squad-negocial.yaml
```

- [ ] **Step 2: Selecionar 4-6 agentes** anotando em `_curado.md`. Critério: papel claro, especialidade que cite legislação real (preferência Lei 14.133/21), diversidade de papéis.

Formato alvo (será JS literal na Task 1.9):

```
agents: [
  { id: 'agente-fiscal-contrato', role: 'Fiscal de Contrato',
    specialty: 'Acompanhamento de execucao (Lei 14.133, art. 117)',
    description: 'Verifica entregas, mede SLA, registra ocorrencias.' },
  // ... 3-5 outros
]
```

Crie `tmp/dominios-extract/contria/_curado.md` com seções `## Agents`, `## Assertions`, `## Scenarios` e popule a primeira seção agora.

### Task 1.6: Curar dados de ContrIA — Assertions

**Files:**
- Read: `tmp/dominios-extract/contria/assertions.md`
- Modify: `tmp/dominios-extract/contria/_curado.md`

- [ ] **Step 1: Ler**

```bash
cat tmp/dominios-extract/contria/assertions.md
```

- [ ] **Step 2: Selecionar 8-12 assertions**, das quais **4 com flag `pipeline: true`** (uma sobre fundamentação legal, uma sobre escopo/objeto, uma sobre disclaimers, uma sobre formatação). Nomes em snake_case ficam como estão (chaves estáveis).

Formato alvo:

```
assertions: [
  { name: 'motivacao_caracterizada_lei14133',
    description: 'Motivacao caracteriza necessidade publica nos termos do art. 11.',
    category: 'fundamentacao', pipeline: true, pipelineOrder: 1 },
  { name: 'objeto_descrito_com_precisao',
    description: 'Objeto descrito de forma clara e suficiente.',
    category: 'escopo', pipeline: true, pipelineOrder: 2 },
  // ...
]
```

### Task 1.7: Curar dados de ContrIA — Scenarios

**Files:**
- Read: `tmp/dominios-extract/contria/cenarios.jsonl`
- Modify: `tmp/dominios-extract/contria/_curado.md`

- [ ] **Step 1: Ler**

```bash
cat tmp/dominios-extract/contria/cenarios.jsonl | head -20
```

- [ ] **Step 2: Selecionar 3 cenários**. Critério: `input` ≤ 200 chars, `esperado` referencia ao menos uma assertion do pipeline, 3 cenários diversos (OK / borderline / institucional).

Formato alvo:

```
scenarios: [
  { id: 'c001', name: 'Caracterizacao de aquisicao direta',
    agent: 'agente-fiscal-contrato',
    input: 'Texto curto de ate 200 chars descrevendo a situacao...',
    expected_keys: ['motivacao_caracterizada_lei14133',
                    'objeto_descrito_com_precisao'],
    output_mock: 'Saida plausivel ~300 chars: A motivacao apresentada caracteriza necessidade publica nos termos do art. 11. Recomendamos: ... [Este texto nao substitui parecer juridico formal.]' },
  // mais 2
]
```

### Task 1.8: Repetir curadoria para LicitIA e PrivacIA

**Files:**
- Read: `tmp/dominios-extract/licitia/*`, `tmp/dominios-extract/privacia/*`
- Create: `tmp/dominios-extract/licitia/_curado.md`, `tmp/dominios-extract/privacia/_curado.md`

- [ ] **Step 1: LicitIA** — repetir Tasks 1.5/1.6/1.7. Mesmos critérios.

- [ ] **Step 2: PrivacIA** — assertions giram em torno de LGPD. Pipeline ideal: `base_legal_lgpd_declarada` → `dados_pessoais_identificados` → `finalidade_especifica` → `disclaimer_lgpd_presente`.

### Task 1.9: Construir o objeto `KAIROS_DATA` completo

**Files:**
- Create: `tmp/dominios-extract/_KAIROS_DATA.js`

- [ ] **Step 1: Compilar o JS literal**

Escrever em `tmp/dominios-extract/_KAIROS_DATA.js` o objeto completo combinando as 3 curadorias:

```
const KAIROS_DATA = {
  featured: {
    contria: {
      id: 'contria',
      name: 'ContrIA',
      fullName: 'Contratos Administrativos',
      regulators: ['Lei 14.133/21', 'TCU', 'CGU'],
      cluster: 'govai',
      iconHint: 'contract',
      agents: [ /* 4-6 da curadoria */ ],
      assertions: [ /* 8-12 da curadoria */ ],
      scenarios: [ /* 3 da curadoria */ ],
      metrics: {
        '1d':  { execs: 627,   tokens: '2.1M',  cost: 'R$ 54',    approval: 99.4 },
        '7d':  { execs: 4521,  tokens: '15.2M', cost: 'R$ 384',   approval: 99.1 },
        '30d': { execs: 18412, tokens: '63.7M', cost: 'R$ 1.601', approval: 99.0 },
      },
      finops: {
        models: [
          { name: 'Claude Sonnet 4.6', cost: 247.10, pct: 64, tokens: '9.7M' },
          { name: 'Claude Haiku 4.5',  cost: 82.40,  pct: 21, tokens: '4.1M' },
          { name: 'GPT-4o',            cost: 54.50,  pct: 15, tokens: '1.4M' },
        ],
        budget: { used: 78, limit: 1500, current: 1170, currency: 'R$' },
        topPrompts: [
          { id: 'p001', label: 'Avaliacao de motivacao Lei 14.133 art. 11',
            model: 'Claude Sonnet 4.6', costSum: 38.20, runs: 412 },
          // 4 outros
        ],
      },
      compliance: {
        categories: ['lei14133'],
        items: [
          /* 5-8 items: { label, regulator, lastCheck: 'pass'|'warning'|'fail', passRate7d: 0..1 } */
        ],
      },
    },
    licitia:  { /* mesma estrutura, com pct e tokens proprios */ },
    privacia: { /* mesma estrutura, categories: ['lgpd'] */ },
  },
  placeholders: [
    { id: 'transparencia', name: 'TransparenciaIA', fullName: 'Transparencia Publica',
      regulators: ['LAI', 'LRF', 'TCE'], eta: '60d',  cluster: 'govai',    iconHint: 'eye' },
    { id: 'ouvidoria',     name: 'OuvidorIA',       fullName: 'Ouvidoria',
      regulators: ['Lei 13.460/17'],     eta: '90d',  cluster: 'govai',    iconHint: 'message' },
    { id: 'servidor',      name: 'ServidorIA',      fullName: 'Servidores Publicos',
      regulators: ['eSocial gov', 'RPPS'],eta: '90d',  cluster: 'govai',    iconHint: 'users' },
    { id: 'saude',         name: 'Saude',           fullName: 'Gestao de Saude',
      regulators: ['CFM', 'ANS', 'ANVISA'],eta: '120d',cluster: 'setorial', iconHint: 'heart' },
    { id: 'educacao',      name: 'Educacao Inclusiva',fullName: 'Educacao Inclusiva',
      regulators: ['LBI', 'BNCC', 'CNE'], eta: '120d',cluster: 'setorial', iconHint: 'school' },
    { id: 'tributario',    name: 'Tributario',      fullName: 'Tributario',
      regulators: ['CTN', 'RFB', 'CARF'], eta: 'roadmap', cluster: 'setorial', iconHint: 'dollar' },
  ],
  global: {
    metrics: {
      '1d':  { execs: 1835,  tokens: '6.3M',   cost: 'R$ 127',   approval: 99.1 },
      '7d':  { execs: 12847, tokens: '43.9M',  cost: 'R$ 891',   approval: 98.7 },
      '30d': { execs: 51284, tokens: '175.4M', cost: 'R$ 3.621', approval: 98.4 },
    },
    finops: {
      models: [
        { name: 'Claude Sonnet 4.6', cost: 107.52, pct: 62, tokens: '28.4M' },
        { name: 'Claude Haiku 4.5',  cost: 36.42,  pct: 21, tokens: '10.8M' },
        { name: 'GPT-4o',            cost: 19.07,  pct: 11, tokens: '3.2M' },
        { name: 'Gemini 2.5 Pro',    cost: 10.41,  pct: 6,  tokens: '1.5M' },
      ],
      budget: { used: 64, limit: 1500, current: 960, currency: 'R$' },
    },
  },
};
```

- [ ] **Step 2: Validar sintaxe**

```bash
node -e "$(cat tmp/dominios-extract/_KAIROS_DATA.js); console.log('featured:', Object.keys(KAIROS_DATA.featured)); console.log('placeholders:', KAIROS_DATA.placeholders.length); console.log('contria agents:', KAIROS_DATA.featured.contria.agents.length);"
```

Expected: `featured: [ 'contria', 'licitia', 'privacia' ]`, `placeholders: 6`, `contria agents: 4` (ou número escolhido).

### Task 1.10: Injetar `KAIROS_DATA` no `index.html`

**Files:**
- Modify: `index.html` — inserir bloco no `<script>` antes da declaração `const slides`

- [ ] **Step 1: Localizar ponto de inserção**

Abrir `index.html`, encontrar a tag `<script>` (~3935). A primeira linha de JS é `const slides = document.querySelectorAll('.slide');`. Inserir `KAIROS_DATA` **antes** dessa linha.

- [ ] **Step 2: Inserir o bloco via Edit**

Usar Edit tool com `old_string`:

```
<script>
const slides = document.querySelectorAll('.slide');
```

e `new_string`:

```
<script>
// KAIROS_DATA - dados curados de dominios (Modo Aplicacao)
// Fonte: github.com/VilelaAI/kairos-dominios-pro (privado)
const KAIROS_DATA = { /* conteudo da Task 1.9 colado aqui inteiro */ };

const slides = document.querySelectorAll('.slide');
```

- [ ] **Step 3: Validar parse no Chrome**

Servir local, abrir `http://localhost:8000`, DevTools Console:

```js
KAIROS_DATA.featured.contria.assertions.length
```

Expected: número entre 8 e 12. Se `Uncaught SyntaxError`, há erro de vírgula/aspas — corrigir.

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat(app): embed curated GovAI domain data (KAIROS_DATA)

Adds agents, assertions, scenarios and metrics for the 3 featured
GovAI domains (ContrIA, LicitIA, PrivacIA) extracted from
kairos-dominios-pro, plus 6 placeholder domains with ETAs.

Co-Authored-By: Claude Opus 4.7 (1M context) <noreply@anthropic.com>"
```

---

## Phase 2 — Store Reativo

### Task 2.1: Adicionar `state`, `setState`, `subscribe`

**Files:**
- Modify: `index.html` — inserir bloco logo após `KAIROS_DATA`, antes de `const slides`.

- [ ] **Step 1: Inserir o store via Edit**

Edit `old_string`:

```
const KAIROS_DATA = { /* ... seu objeto literal aqui ... */ };

const slides = document.querySelectorAll('.slide');
```

e `new_string`: o mesmo bloco do `KAIROS_DATA` seguido de:

```js
// STATE STORE - reativo, vanilla, sem deps
const state = {
  currentDomain: null,       // 'contria' | 'licitia' | 'privacia' | null
  currentScreen: 'overview', // 'overview'|'catalog'|'agents'|'assertions'|'playground'|'observability'|'finops'|'compliance'
  period: '7d',              // '1d' | '7d' | '30d'
  drillDown: null,           // { type, payload } | null
  ticker: { recentTraces: [], lastTick: Date.now(), paused: false },
  filters: {
    catalog:    { regulator: 'all', query: '' },
    agents:     { role: 'all' },
    assertions: { status: 'all' },
    compliance: { category: 'lei14133' },
  },
};

const __subscribers = new Set();
function setState(patch) {
  Object.assign(state, patch);
  __subscribers.forEach(fn => { try { fn(state); } catch (e) { console.error('[setState]', e); } });
}
function subscribe(fn) { __subscribers.add(fn); return () => __subscribers.delete(fn); }

function activeDomainData() {
  return state.currentDomain ? KAIROS_DATA.featured[state.currentDomain] : null;
}
function activeMetrics() {
  const d = activeDomainData();
  return d ? d.metrics[state.period] : KAIROS_DATA.global.metrics[state.period];
}

const slides = document.querySelectorAll('.slide');
```

- [ ] **Step 2: Smoke test no console**

```js
state.currentDomain      // null
setState({ currentDomain: 'contria' });
state.currentDomain      // 'contria'
activeMetrics()          // objeto com execs/tokens/cost/approval (7d)
activeDomainData().name  // 'ContrIA'
setState({ currentDomain: null });
```

Tudo passa, console limpo.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat(app): add reactive state store + subscribe/setState

Co-Authored-By: Claude Opus 4.7 (1M context) <noreply@anthropic.com>"
```

### Task 2.2: Migrar `appNavigate` para usar o store

**Files:**
- Modify: `index.html` ~3986-4043 (função `appNavigate`, listeners)

- [ ] **Step 1: Adaptar `appNavigate`**

Edit: localizar a primeira linha do corpo de `function appNavigate(screen) {` e inserir `setState({ currentScreen: screen });` como primeira instrução. O resto da função permanece.

- [ ] **Step 2: Adaptar listener de `.domain-card` (provisório)**

Localizar:

```js
document.querySelectorAll('#kairosApp .domain-card').forEach(card => {
  card.addEventListener('click', () => {
    appNavigate('playground');
    pgLoad();
  });
});
```

Substituir por:

```js
document.querySelectorAll('#kairosApp .domain-card[data-domain]').forEach(card => {
  card.addEventListener('click', () => {
    const id = card.dataset.domain;
    const legacy = { trabalhista: 'contria', educacao: 'contria', juridico: 'licitia' };
    const featuredId = legacy[id] || (KAIROS_DATA.featured[id] ? id : null);
    if (featuredId) setState({ currentDomain: featuredId });
    appNavigate('playground');
    pgLoad();
  });
});
```

Nota: este mapping legacy é descartado na Task 4.1.

- [ ] **Step 3: Smoke test**

`A` → clique em qualquer card → vai pro Playground. Console: `state.currentDomain` tem valor (ou null para cards sem match).

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "refactor(app): route navigation/domain selection through setState

Co-Authored-By: Claude Opus 4.7 (1M context) <noreply@anthropic.com>"
```

---

## Phase 3 — Drill Panel

### Task 3.1: HTML + CSS do `<aside id="drillPanel">`

**Files:**
- Modify: `index.html` — adicionar HTML do drill panel dentro de `#kairosApp` antes do fechamento de `.studio` (~linha 2921). Adicionar CSS no bloco `<style>` (~linha 1825).

- [ ] **Step 1: Inserir HTML do drill panel**

Localizar a linha exata `</div>` que fecha `<div class="studio">`. Inserir, **antes** desse fechamento:

```html
<div id="drillBackdrop"></div>
<aside id="drillPanel" role="dialog" aria-modal="true" aria-hidden="true">
  <header class="drill-head">
    <div>
      <div class="drill-eyebrow" id="drillEyebrow">Detalhe</div>
      <h2 class="drill-title" id="drillTitle">—</h2>
    </div>
    <button class="drill-close" id="drillCloseBtn" aria-label="Fechar (Esc)">
      <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><line x1="18" y1="6" x2="6" y2="18"/><line x1="6" y1="6" x2="18" y2="18"/></svg>
    </button>
  </header>
  <div class="drill-body" id="drillBody"></div>
  <footer class="drill-foot" id="drillFoot"></footer>
</aside>
```

- [ ] **Step 2: Inserir CSS do drill panel**

Adicionar antes da seção `TELA: OVERVIEW` no bloco `<style>`:

```css
#drillBackdrop {
  position: fixed; inset: 0;
  background: rgba(7, 8, 12, 0.55);
  backdrop-filter: blur(2px);
  z-index: 250;
  opacity: 0; pointer-events: none;
  transition: opacity 0.2s ease;
}
body.drill-open #drillBackdrop { opacity: 1; pointer-events: auto; }

#drillPanel {
  position: fixed; top: 0; right: 0; bottom: 0;
  width: 520px; max-width: 90vw;
  background: #0E1019;
  border-left: 1px solid #1F2230;
  box-shadow: -20px 0 60px rgba(0, 0, 0, 0.4);
  z-index: 260;
  display: flex; flex-direction: column;
  transform: translateX(100%);
  transition: transform 0.28s cubic-bezier(0.22, 0.61, 0.36, 1);
  font-family: 'Inter', system-ui, sans-serif;
  color: #E5E7EB;
}
body.drill-open #drillPanel { transform: translateX(0); }

.drill-head {
  display: flex; align-items: flex-start; justify-content: space-between;
  padding: 24px 28px 18px;
  border-bottom: 1px solid #1F2230;
  gap: 12px;
}
.drill-eyebrow {
  font-family: 'Space Grotesk', sans-serif;
  font-size: 11px; letter-spacing: 0.14em;
  text-transform: uppercase;
  color: hsl(var(--primary));
  font-weight: 600; margin-bottom: 6px;
}
.drill-title {
  font-family: 'Space Grotesk', sans-serif;
  font-size: 22px; font-weight: 700;
  color: #fff; line-height: 1.2; letter-spacing: -0.01em;
}
.drill-close {
  background: rgba(255, 255, 255, 0.04);
  border: 1px solid #252836;
  color: #9CA3AF;
  border-radius: 8px;
  width: 36px; height: 36px;
  display: flex; align-items: center; justify-content: center;
  cursor: pointer; flex-shrink: 0;
  transition: all 0.15s;
}
.drill-close:hover { background: rgba(255, 255, 255, 0.08); color: #fff; }
.drill-body { flex: 1; overflow-y: auto; padding: 22px 28px; font-size: 14px; line-height: 1.55; }
.drill-body::-webkit-scrollbar { width: 8px; }
.drill-body::-webkit-scrollbar-track { background: transparent; }
.drill-body::-webkit-scrollbar-thumb { background: #252836; border-radius: 4px; }
.drill-foot {
  padding: 16px 28px 20px;
  border-top: 1px solid #1F2230;
  display: flex; gap: 10px; justify-content: flex-end;
}
.drill-foot:empty { display: none; }

.drill-section { margin-bottom: 22px; }
.drill-section:last-child { margin-bottom: 0; }
.drill-section-title {
  font-family: 'Space Grotesk', sans-serif;
  font-size: 12px; font-weight: 600;
  letter-spacing: 0.08em; text-transform: uppercase;
  color: #6B7280; margin-bottom: 10px;
}
.drill-kv { display: grid; grid-template-columns: 140px 1fr; gap: 8px 16px; font-size: 13px; }
.drill-kv dt { color: #6B7280; }
.drill-kv dd { color: #E5E7EB; }
.drill-list { display: flex; flex-direction: column; gap: 8px; }
.drill-list-item {
  background: rgba(255, 255, 255, 0.02);
  border: 1px solid #1F2230;
  border-radius: 8px;
  padding: 12px 14px;
}
.drill-list-item .di-title { font-weight: 600; color: #fff; margin-bottom: 4px; font-size: 13px; }
.drill-list-item .di-meta { font-size: 12px; color: #9CA3AF; }
.drill-tag {
  display: inline-flex; align-items: center; gap: 6px;
  padding: 3px 10px;
  background: rgba(36, 113, 245, 0.12);
  border: 1px solid hsl(var(--primary) / 0.3);
  border-radius: 999px;
  font-size: 11px; font-weight: 500;
  color: hsl(var(--primary));
  letter-spacing: 0.03em;
}
.drill-tag.success { background: rgba(34, 197, 94, 0.12); border-color: rgba(34, 197, 94, 0.3); color: #22C55E; }
.drill-tag.warn    { background: rgba(245, 158, 11, 0.12); border-color: rgba(245, 158, 11, 0.3); color: #F59E0B; }
.drill-tag.fail    { background: rgba(239, 68, 68, 0.12);  border-color: rgba(239, 68, 68, 0.3);  color: #EF4444; }
```

- [ ] **Step 3: Validar visualmente**

`A` para Modo Aplicação. No console:

```js
document.body.classList.add('drill-open')
```

Expected: painel slide-in entra da direita; backdrop cinza aparece. Reverter: `document.body.classList.remove('drill-open')`.

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat(app): drill panel HTML + slide-in CSS

Co-Authored-By: Claude Opus 4.7 (1M context) <noreply@anthropic.com>"
```

### Task 3.2: Renderer do drill panel com switch por `type`

**Files:**
- Modify: `index.html` `<script>` — adicionar bloco antes do comentário `MODO APLICAÇÃO` (~3974)

- [ ] **Step 1: Adicionar renderer + helpers**

Edit: localizar a linha do comentário `// MODO APLICAÇÃO` e inserir antes dela:

```js
// DRILL PANEL
function openDrill(type, payload) { setState({ drillDown: { type, payload } }); }
function closeDrill() { setState({ drillDown: null }); }

function renderDrillPanel(s) {
  const dp = document.getElementById('drillPanel');
  const eyebrow = document.getElementById('drillEyebrow');
  const title   = document.getElementById('drillTitle');
  const body    = document.getElementById('drillBody');
  const foot    = document.getElementById('drillFoot');

  if (!s.drillDown) {
    document.body.classList.remove('drill-open');
    dp.setAttribute('aria-hidden', 'true');
    return;
  }
  const { type, payload } = s.drillDown;
  body.textContent = ''; foot.textContent = '';

  switch (type) {
    case 'domain':          renderDomainDrill(payload, eyebrow, title, body, foot); break;
    case 'placeholder':     renderPlaceholderDrill(payload, eyebrow, title, body, foot); break;
    case 'agent':           renderAgentDrill(payload, eyebrow, title, body, foot); break;
    case 'assertion':       renderAssertionDrill(payload, eyebrow, title, body, foot); break;
    case 'trace':           renderTraceDrill(payload, eyebrow, title, body, foot); break;
    case 'model':           renderModelDrill(payload, eyebrow, title, body, foot); break;
    case 'budget':          renderBudgetDrill(payload, eyebrow, title, body, foot); break;
    case 'compliance-item': renderComplianceItemDrill(payload, eyebrow, title, body, foot); break;
    default:
      eyebrow.textContent = 'Detalhe';
      title.textContent = '—';
      body.textContent = 'Conteudo nao disponivel.';
  }
  document.body.classList.add('drill-open');
  dp.setAttribute('aria-hidden', 'false');
}
subscribe(renderDrillPanel);

// Domain drill: cabecalho com regulators + listas de agentes/assertions
function renderDomainDrill(d, eyebrow, title, body, foot) {
  eyebrow.textContent = 'Dominio Featured';
  title.textContent = d.fullName;
  const agentsListHtml = d.agents.map(function (a) {
    return '<div class="drill-list-item">'
      + '<div class="di-title">' + a.role + '</div>'
      + '<div class="di-meta">' + a.specialty + '</div>'
      + '</div>';
  }).join('');
  const asListHtml = d.assertions.slice(0, 5).map(function (a) {
    return '<div class="drill-list-item">'
      + '<div class="di-title"><code>' + a.name + '</code></div>'
      + '<div class="di-meta">' + a.description + '</div>'
      + '</div>';
  }).join('');
  body.innerHTML =
    '<div class="drill-section">'
    + '<dl class="drill-kv">'
    + '<dt>Codinome</dt><dd><strong>' + d.name + '</strong></dd>'
    + '<dt>Regulacao</dt><dd>' + d.regulators.join(' &middot; ') + '</dd>'
    + '<dt>Agentes</dt><dd>' + d.agents.length + '</dd>'
    + '<dt>Assertions</dt><dd>' + d.assertions.length + '</dd>'
    + '<dt>Cenarios</dt><dd>' + d.scenarios.length + '</dd>'
    + '</dl>'
    + '</div>'
    + '<div class="drill-section">'
    + '<div class="drill-section-title">Agentes (' + d.agents.length + ')</div>'
    + '<div class="drill-list">' + agentsListHtml + '</div>'
    + '</div>'
    + '<div class="drill-section">'
    + '<div class="drill-section-title">Top assertions</div>'
    + '<div class="drill-list">' + asListHtml + '</div>'
    + '</div>';
  foot.innerHTML =
    '<button class="app-btn secondary" id="drillCancelBtn">Fechar</button>'
    + '<button class="app-btn" id="drillRunBtn"><svg width="14" height="14" viewBox="0 0 24 24" fill="currentColor"><polygon points="6 4 20 12 6 20 6 4"/></svg> Executar no Playground</button>';
  document.getElementById('drillCancelBtn').addEventListener('click', closeDrill);
  document.getElementById('drillRunBtn').addEventListener('click', function () {
    setState({ currentDomain: d.id, drillDown: null });
    appNavigate('playground');
    pgLoad();
  });
}

// Placeholder drill: ETA + contato
function renderPlaceholderDrill(d, eyebrow, title, body, foot) {
  eyebrow.textContent = 'Em onboarding';
  title.textContent = d.fullName;
  body.innerHTML =
    '<div class="drill-section">'
    + '<dl class="drill-kv">'
    + '<dt>Codinome</dt><dd><strong>' + d.name + '</strong></dd>'
    + '<dt>Regulacao</dt><dd>' + d.regulators.join(' &middot; ') + '</dd>'
    + '<dt>ETA</dt><dd><span class="drill-tag warn">' + d.eta + '</span></dd>'
    + '<dt>Cluster</dt><dd>' + (d.cluster === 'govai' ? 'GovAI' : 'Setorial') + '</dd>'
    + '</dl>'
    + '</div>'
    + '<div class="drill-section">'
    + '<div class="drill-section-title">Roadmap</div>'
    + '<p>O squad de ' + d.name + ' esta em validacao com '
    + (d.cluster === 'govai' ? 'orgaos parceiros' : 'clientes-piloto')
    + '. Cobertura inicial: agentes negociais + assertions de fundamentacao legal + cenarios representativos.</p>'
    + '</div>'
    + '<div class="drill-section">'
    + '<div class="drill-section-title">Quer acelerar?</div>'
    + '<p>Demanda priorizada: <a href="mailto:contato@vilela.ai" style="color:hsl(var(--primary))">contato@vilela.ai</a></p>'
    + '</div>';
  foot.innerHTML = '<button class="app-btn secondary" id="drillCancelBtn">Fechar</button>';
  document.getElementById('drillCancelBtn').addEventListener('click', closeDrill);
}

// Stubs (substituidos por tasks especificas; mantem o switch funcionando agora)
function renderAgentDrill(a, eyebrow, title, body, foot)          { eyebrow.textContent = 'Agente';     title.textContent = a.role || '—';  body.textContent = a.description || ''; }
function renderAssertionDrill(a, eyebrow, title, body, foot)      { eyebrow.textContent = 'Assertion';  title.textContent = a.name || '—';  body.textContent = a.description || ''; }
function renderTraceDrill(t, eyebrow, title, body, foot)          { eyebrow.textContent = 'Trace';      title.textContent = t.id || '—';    body.textContent = JSON.stringify(t); }
function renderModelDrill(m, eyebrow, title, body, foot)          { eyebrow.textContent = 'Modelo';     title.textContent = m.name || '—';  body.textContent = m.tokens || ''; }
function renderBudgetDrill(b, eyebrow, title, body, foot)         { eyebrow.textContent = 'Budget';     title.textContent = 'Orcamento';    body.textContent = (b.used || 0) + '% do orcamento utilizado.'; }
function renderComplianceItemDrill(c, eyebrow, title, body, foot) { eyebrow.textContent = 'Compliance'; title.textContent = c.label || '—'; body.textContent = c.regulator || ''; }
```

**Por que `.innerHTML` aqui é seguro:** veja "Nota de Segurança" no início deste plano. Todo conteúdo interpolado vem de `KAIROS_DATA` curado por nós ou de objetos construídos pelo próprio código. `d.fullName`, `d.regulators`, etc são strings controladas no build.

- [ ] **Step 2: Smoke test**

`A` → console:

```js
openDrill('domain', KAIROS_DATA.featured.contria)
```

Expected: painel abre com info de ContrIA, 2 botões no footer. "Executar no Playground" navega.

```js
closeDrill()
openDrill('placeholder', KAIROS_DATA.placeholders[0])
```

Expected: drill placeholder com TransparenciaIA.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat(app): drill panel renderer with type dispatch

Co-Authored-By: Claude Opus 4.7 (1M context) <noreply@anthropic.com>"
```

### Task 3.3: Listeners de fechamento + atalhos `T`/`D`/`Esc`

**Files:**
- Modify: `index.html` — handler `keydown` (~3948) + listeners do drill

- [ ] **Step 1: Hook do botão X e backdrop**

Após o bloco do drill (Task 3.2), adicionar:

```js
document.getElementById('drillCloseBtn').addEventListener('click', closeDrill);
document.getElementById('drillBackdrop').addEventListener('click', closeDrill);
```

- [ ] **Step 2: Estender keydown em app-mode**

Localizar o bloco `if (document.body.classList.contains('app-mode')) { if (e.key === 'Escape') toggleApp(); return; }` (~3950). Substituir todo o `if` por:

```js
if (document.body.classList.contains('app-mode')) {
  if (e.key === 'Escape') {
    if (state.drillDown) closeDrill(); else toggleApp();
    return;
  }
  if (e.key === 'D' || e.key === 'd') {
    if (state.drillDown) closeDrill();
    return;
  }
  if (e.key === 'T' || e.key === 't') {
    setState({ ticker: Object.assign({}, state.ticker, { paused: !state.ticker.paused }) });
    showAppToast(state.ticker.paused ? 'Ticker pausado' : 'Ticker retomado');
    return;
  }
  return;
}
```

- [ ] **Step 3: Adicionar `showAppToast` no final do `<script>` (antes de `fitScale`)**

```js
let __toastTimer = null;
function showAppToast(msg) {
  let toast = document.getElementById('appToast');
  if (!toast) {
    toast = document.createElement('div');
    toast.id = 'appToast';
    document.body.appendChild(toast);
  }
  toast.textContent = msg;
  toast.classList.add('visible');
  clearTimeout(__toastTimer);
  __toastTimer = setTimeout(function () { toast.classList.remove('visible'); }, 1500);
}
```

CSS (no bloco `<style>`, após `.app-mode-hint`):

```css
#appToast {
  position: fixed;
  bottom: 24px; left: 50%;
  transform: translate(-50%, 12px);
  z-index: 400;
  background: rgba(17, 19, 28, 0.96);
  border: 1px solid #252836;
  color: #E5E7EB;
  padding: 10px 18px;
  border-radius: 999px;
  font-family: 'Inter', system-ui, sans-serif;
  font-size: 13px; font-weight: 500;
  backdrop-filter: blur(8px);
  opacity: 0; pointer-events: none;
  transition: opacity 0.2s, transform 0.2s;
}
#appToast.visible { opacity: 1; transform: translate(-50%, 0); }
```

- [ ] **Step 4: Smoke test (10 cenários)**

1. `A` → entra Modo App
2. Console: `openDrill('domain', KAIROS_DATA.featured.contria)` → drill abre
3. `Esc` → drill fecha (continua em Modo App)
4. `Esc` de novo → sai do Modo App
5. `A` → entra
6. `T` → toast "Ticker pausado"
7. `T` → "Ticker retomado"
8. `D` com drill fechado → nada
9. `openDrill(...)`, `D` → fecha
10. Drill aberto, clique no backdrop → fecha

Todos passando, console limpo.

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat(app): drill close handlers (Esc/D/X/backdrop) + toast helper

Co-Authored-By: Claude Opus 4.7 (1M context) <noreply@anthropic.com>"
```

---

## Phase 4 — Tela Catalog v2

### Task 4.1: Regenerar cards do catálogo a partir de `KAIROS_DATA`

**Files:**
- Modify: `index.html` — região do `<div class="app-screen" data-screen="catalog">` (~2982-3103)
- Modify: `index.html` `<script>` — adicionar `renderCatalog`

- [ ] **Step 1: Substituir o conteúdo do grid**

Edit: localizar `<div class="domain-grid">` dentro de `data-screen="catalog"` (com seus 9 cards estáticos). Substituir o conteúdo do grid por:

```html
<div class="domain-grid" id="catalogGrid">
  <!-- preenchido por renderCatalog -->
</div>
```

- [ ] **Step 2: Adicionar `renderCatalog` no `<script>`**

Adicionar antes do bloco PLAYGROUND:

```js
// CATALOG
const DOMAIN_ICONS = {
  contract:  '<path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z"/><polyline points="14 2 14 8 20 8"/><line x1="8" y1="13" x2="16" y2="13"/><line x1="8" y1="17" x2="16" y2="17"/>',
  document:  '<path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z"/><polyline points="14 2 14 8 20 8"/>',
  shield:    '<path d="M12 22s8-4 8-10V5l-8-3-8 3v7c0 6 8 10 8 10z"/>',
  eye:       '<path d="M1 12s4-8 11-8 11 8 11 8-4 8-11 8S1 12 1 12z"/><circle cx="12" cy="12" r="3"/>',
  message:   '<path d="M21 15a2 2 0 0 1-2 2H7l-4 4V5a2 2 0 0 1 2-2h14a2 2 0 0 1 2 2z"/>',
  users:     '<path d="M17 21v-2a4 4 0 0 0-4-4H5a4 4 0 0 0-4 4v2"/><circle cx="9" cy="7" r="4"/>',
  heart:     '<path d="M20.84 4.61a5.5 5.5 0 0 0-7.78 0L12 5.67l-1.06-1.06a5.5 5.5 0 0 0-7.78 7.78l1.06 1.06L12 21.23l7.78-7.78 1.06-1.06a5.5 5.5 0 0 0 0-7.78z"/>',
  school:    '<path d="M22 10v6M2 10l10-5 10 5-10 5z"/><path d="M6 12v5c3 3 9 3 12 0v-5"/>',
  dollar:    '<line x1="12" y1="1" x2="12" y2="23"/><path d="M17 5H9.5a3.5 3.5 0 0 0 0 7h5a3.5 3.5 0 0 1 0 7H6"/>',
};
function domainIconSVG(hint) {
  return '<svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">' + (DOMAIN_ICONS[hint] || DOMAIN_ICONS.document) + '</svg>';
}

function renderCatalog(s) {
  const grid = document.getElementById('catalogGrid');
  if (!grid) return;

  const featured = Object.values(KAIROS_DATA.featured);
  const placeholders = KAIROS_DATA.placeholders;
  const query = s.filters.catalog.query.toLowerCase().trim();
  const regulator = s.filters.catalog.regulator;

  function matches(d, isFeatured) {
    if (query) {
      const hay = (d.name + ' ' + d.fullName + ' ' + (d.regulators || []).join(' ')).toLowerCase();
      if (!hay.includes(query)) return false;
    }
    if (regulator === 'all') return true;
    if (regulator === 'lei14133')   return (d.regulators || []).some(r => r.includes('14.133'));
    if (regulator === 'lgpd')       return (d.regulators || []).some(r => r.toLowerCase().includes('lgpd'));
    if (regulator === 'govai')      return d.cluster === 'govai';
    if (regulator === 'onboarding') return !isFeatured;
    return true;
  }

  const featuredCards = featured.filter(d => matches(d, true)).map(d => {
    return '<article class="domain-card featured" data-domain-id="' + d.id + '" data-featured="true">'
      + '<div class="domain-card-head">'
      + '<div class="domain-icon">' + domainIconSVG(d.iconHint) + '</div>'
      + '<span class="domain-status">Ativo</span>'
      + '</div>'
      + '<div class="domain-name">' + d.fullName + '</div>'
      + '<div class="domain-regulator">' + d.regulators.join(' &middot; ') + '</div>'
      + '<div class="domain-metrics">'
      + '<div class="domain-metric"><div class="domain-metric-val">' + d.agents.length + '</div><div class="domain-metric-key">Agentes</div></div>'
      + '<div class="domain-metric"><div class="domain-metric-val">' + d.assertions.length + '</div><div class="domain-metric-key">Assertions</div></div>'
      + '<div class="domain-metric"><div class="domain-metric-val">' + d.scenarios.length + '</div><div class="domain-metric-key">Cenarios</div></div>'
      + '</div>'
      + '</article>';
  }).join('');

  const placeholderCards = placeholders.filter(d => matches(d, false)).map(d => {
    return '<article class="domain-card placeholder" data-domain-id="' + d.id + '" data-featured="false">'
      + '<div class="domain-card-head">'
      + '<div class="domain-icon">' + domainIconSVG(d.iconHint) + '</div>'
      + '<span class="domain-status onboarding">Onboarding</span>'
      + '</div>'
      + '<div class="domain-name">' + d.fullName + '</div>'
      + '<div class="domain-regulator">' + d.regulators.join(' &middot; ') + '</div>'
      + '<div class="domain-eta">ETA: <strong>' + d.eta + '</strong></div>'
      + '</article>';
  }).join('');

  grid.innerHTML = featuredCards + placeholderCards;

  grid.querySelectorAll('.domain-card').forEach(card => {
    card.addEventListener('click', () => {
      const id = card.dataset.domainId;
      if (card.dataset.featured === 'true') {
        openDrill('domain', KAIROS_DATA.featured[id]);
      } else {
        openDrill('placeholder', KAIROS_DATA.placeholders.find(p => p.id === id));
      }
    });
  });
}
subscribe(renderCatalog);
renderCatalog(state); // primeiro render
```

- [ ] **Step 3: CSS para `.placeholder` e `.domain-eta`**

Adicionar no bloco `<style>` antes da seção do drill:

```css
#kairosApp .domain-card.placeholder {
  background: rgba(255, 255, 255, 0.015);
  border-style: dashed;
  opacity: 0.85;
}
#kairosApp .domain-card.placeholder:hover {
  opacity: 1;
  border-style: solid;
  border-color: rgba(245, 158, 11, 0.4);
  box-shadow: 0 8px 24px rgba(245, 158, 11, 0.08);
}
#kairosApp .domain-status.onboarding {
  background: rgba(245, 158, 11, 0.12);
  border: 1px solid rgba(245, 158, 11, 0.3);
  color: #F59E0B;
}
#kairosApp .domain-eta {
  font-size: 12px;
  color: #6B7280;
  margin-top: 8px;
  padding-top: 10px;
  border-top: 1px solid #1F2230;
}
#kairosApp .domain-eta strong { color: #F59E0B; }
```

- [ ] **Step 4: Smoke test**

`A` → Catálogo. Expected: 3 featured (ContrIA, LicitIA, PrivacIA) com glow + 6 placeholders com badge "Onboarding" e ETA. Clicar ContrIA → drill abre. "Executar no Playground" navega. Clicar TransparenciaIA → drill placeholder.

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat(app): catalog v2 - cards rendered from KAIROS_DATA

Co-Authored-By: Claude Opus 4.7 (1M context) <noreply@anthropic.com>"
```

### Task 4.2: Filter chips + search funcionais

**Files:**
- Modify: `index.html` — adicionar UI de filtros + listeners

- [ ] **Step 1: Inserir UI**

No HTML da tela catalog, após o `<div class="page-head">` e ANTES do `<div class="domain-grid" id="catalogGrid">`, inserir:

```html
<div class="catalog-filters">
  <div class="catalog-search">
    <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><circle cx="11" cy="11" r="8"/><line x1="21" y1="21" x2="16.65" y2="16.65"/></svg>
    <input type="text" id="catalogSearch" placeholder="Buscar por nome ou regulador..." autocomplete="off">
  </div>
  <div class="catalog-chips" id="catalogChips">
    <button class="ag-chip active" data-filter="all">Todos</button>
    <button class="ag-chip" data-filter="govai">GovAI</button>
    <button class="ag-chip" data-filter="lei14133">Lei 14.133</button>
    <button class="ag-chip" data-filter="lgpd">LGPD</button>
    <button class="ag-chip" data-filter="onboarding">Em onboarding</button>
  </div>
</div>
```

- [ ] **Step 2: CSS dos filtros**

```css
.catalog-filters {
  display: flex; align-items: center; justify-content: space-between;
  gap: 16px; margin-bottom: 20px; flex-wrap: wrap;
}
.catalog-search {
  display: flex; align-items: center; gap: 10px;
  background: #0B0C12;
  border: 1px solid #252836;
  border-radius: 10px;
  padding: 8px 14px;
  min-width: 320px;
  color: #6B7280;
}
.catalog-search:focus-within { border-color: hsl(var(--primary) / 0.5); color: hsl(var(--primary)); }
.catalog-search input {
  background: none; border: none;
  color: #E5E7EB;
  font-family: 'Inter', system-ui, sans-serif;
  font-size: 13px; outline: none; flex: 1;
}
.catalog-search input::placeholder { color: #4A5160; }
.catalog-chips { display: flex; gap: 8px; flex-wrap: wrap; }
```

Regras `.ag-chip` já existem; confirmar com `grep -n "\\.ag-chip " index.html`. Caso ausentes:

```css
.ag-chip {
  background: #0B0C12;
  border: 1px solid #252836;
  border-radius: 999px;
  padding: 6px 14px;
  font-family: 'Inter', system-ui, sans-serif;
  font-size: 12px;
  color: #9CA3AF;
  cursor: pointer;
  transition: all 0.15s;
}
.ag-chip:hover { color: #E5E7EB; border-color: hsl(var(--primary) / 0.4); }
.ag-chip.active { background: hsl(var(--primary) / 0.15); border-color: hsl(var(--primary) / 0.6); color: hsl(var(--primary)); }
```

- [ ] **Step 3: Wire listeners**

Logo após `renderCatalog(state)`:

```js
document.getElementById('catalogSearch').addEventListener('input', (e) => {
  setState({ filters: Object.assign({}, state.filters, { catalog: Object.assign({}, state.filters.catalog, { query: e.target.value }) }) });
});
document.querySelectorAll('#catalogChips .ag-chip').forEach(chip => {
  chip.addEventListener('click', () => {
    document.querySelectorAll('#catalogChips .ag-chip').forEach(c => c.classList.remove('active'));
    chip.classList.add('active');
    setState({ filters: Object.assign({}, state.filters, { catalog: Object.assign({}, state.filters.catalog, { regulator: chip.dataset.filter }) }) });
  });
});
```

- [ ] **Step 4: Smoke test**

`A` → Catálogo:
- "lei" na busca → filtra ContrIA + LicitIA
- Limpar; chip "GovAI" → 6 cards
- Chip "Em onboarding" → 6 placeholders
- Chip "Todos" → 9 cards

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat(app): catalog filters (search + chips) functional

Co-Authored-By: Claude Opus 4.7 (1M context) <noreply@anthropic.com>"
```

---

## Phase 5 — Tela Playground v2

### Task 5.1: Scenario picker dinâmico + pipeline por domínio

**Files:**
- Modify: `index.html` — tela playground

- [ ] **Step 1: Adicionar scenario picker no page-head**

Localizar `<div class="page-head">` em `data-screen="playground"`. Dentro do `<div>` que contém `page-title`/`page-sub`, adicionar (após o `<div class="page-sub" id="pgSub">`):

```html
<div class="pg-scenario-picker" id="pgScenarioPicker" style="display:none">
  <label>Cenario:</label>
  <select id="pgScenarioSelect"></select>
</div>
```

CSS:

```css
.pg-scenario-picker {
  margin-top: 10px;
  display: flex; align-items: center; gap: 10px;
  font-size: 13px;
}
.pg-scenario-picker label { color: #6B7280; }
.pg-scenario-picker select {
  background: #0B0C12;
  border: 1px solid #252836;
  border-radius: 6px;
  color: #E5E7EB;
  padding: 6px 28px 6px 12px;
  font-family: 'Inter', system-ui, sans-serif;
  font-size: 13px;
  cursor: pointer;
}
.pg-scenario-picker select:focus { outline: none; border-color: hsl(var(--primary) / 0.5); }
```

- [ ] **Step 2: Substituir `pgLoad`**

Localizar `function pgLoad() { ... }`. Substituir corpo completo por:

```js
function pgLoad() {
  const d = activeDomainData();
  const picker = document.getElementById('pgScenarioPicker');
  const select = document.getElementById('pgScenarioSelect');

  if (!d) {
    document.getElementById('pgSub').textContent = 'Nenhum dominio ativo. Volte ao Catalogo.';
    picker.style.display = 'none';
    document.getElementById('pgEmpty').style.display = 'flex';
    document.getElementById('pgLoaded').style.display = 'none';
    document.getElementById('pgRun').style.display = 'none';
    document.getElementById('pgReset').style.display = 'none';
    document.getElementById('pgGoObserv').style.display = 'none';
    return;
  }

  select.innerHTML = d.scenarios.map(s => '<option value="' + s.id + '">' + s.name + '</option>').join('');
  picker.style.display = 'flex';

  document.getElementById('pgSub').textContent = 'Agente ' + (d.agents[0].id || d.agents[0].role) + ' carregado · cenario "' + d.scenarios[0].name + '" · clique Executar';

  document.getElementById('pgEmpty').style.display = 'none';
  document.getElementById('pgLoaded').style.display = 'grid';
  document.getElementById('pgRun').style.display = 'inline-flex';
  document.getElementById('pgReset').style.display = 'none';
  document.getElementById('pgGoObserv').style.display = 'none';

  document.querySelectorAll('#pgPipeline .validation-step').forEach(s => s.classList.remove('running', 'pass'));
  document.getElementById('pgOutput').classList.remove('visible');
  document.getElementById('pgSummary').classList.remove('visible');

  renderPgPipeline(d);
}

function renderPgPipeline(d) {
  const pipelineSteps = d.assertions.filter(a => a.pipeline).sort((a, b) => (a.pipelineOrder || 0) - (b.pipelineOrder || 0)).slice(0, 4);
  const pipelineEl = document.getElementById('pgPipeline');
  if (!pipelineEl) return;

  pipelineEl.innerHTML = pipelineSteps.map((a, i) =>
    '<div class="validation-step" data-assertion="' + a.name + '">'
    + '<div class="val-check"><svg width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="3"><polyline points="20 6 9 17 4 12"/></svg></div>'
    + '<div class="val-content"><div class="val-name">' + (i + 1) + '. ' + a.description + '</div><code class="val-meta">' + a.name + '</code></div>'
    + '<span class="val-step-time">—</span>'
    + '</div>'
  ).join('');
}
```

- [ ] **Step 3: CSS para `val-content`/`val-name`/`val-meta` (se ausente)**

```css
#kairosApp .validation-step {
  display: flex; align-items: center; gap: 14px;
  padding: 14px 18px;
  background: #0B0C12;
  border: 1px solid #1F2230;
  border-radius: 10px;
}
#kairosApp .val-content { flex: 1; }
#kairosApp .val-name { font-size: 13px; color: #E5E7EB; font-weight: 500; margin-bottom: 4px; }
#kairosApp .val-meta { font-family: 'JetBrains Mono', monospace; font-size: 11px; color: #6B7280; }
```

Verificar com `grep -n "val-content\\|val-name\\b" index.html`. Adicionar apenas o que faltar.

- [ ] **Step 4: Listener do scenario picker**

Adicionar próximo aos outros listeners do playground:

```js
document.getElementById('pgScenarioSelect').addEventListener('change', (e) => {
  const d = activeDomainData();
  if (!d) return;
  const scenarioId = e.target.value;
  const sc = d.scenarios.find(s => s.id === scenarioId);
  document.getElementById('pgSub').textContent = 'Cenario "' + sc.name + '" pronto · clique Executar';
  document.querySelectorAll('#pgPipeline .validation-step').forEach(s => s.classList.remove('running', 'pass'));
  document.getElementById('pgOutput').classList.remove('visible');
  document.getElementById('pgSummary').classList.remove('visible');
});
```

- [ ] **Step 5: Smoke test**

Catálogo → drill ContrIA → "Executar no Playground". Expected:
- Dropdown "Cenário:" com 3 cenários de ContrIA
- 4 assertion-steps de ContrIA (não genéricos M.S.A.)
- Trocar cenário → sub-text atualiza

- [ ] **Step 6: Commit**

```bash
git add index.html
git commit -m "feat(app): playground v2 - scenario picker + per-domain pipeline

Co-Authored-By: Claude Opus 4.7 (1M context) <noreply@anthropic.com>"
```

### Task 5.2: Output real + emissão de trace

**Files:**
- Modify: `index.html` — substituir `function pgRun`

- [ ] **Step 1: Substituir `pgRun`**

Localizar `async function pgRun() { ... }` e substituir corpo completo por:

```js
async function pgRun() {
  const d = activeDomainData();
  if (!d) return;
  const scenarioId = document.getElementById('pgScenarioSelect').value;
  const scenario = d.scenarios.find(s => s.id === scenarioId) || d.scenarios[0];

  const runBtn = document.getElementById('pgRun');
  runBtn.disabled = true;
  runBtn.innerHTML = '<span class="app-spinner"></span> Executando...';

  document.getElementById('pgSub').textContent = 'Pipeline anti-alucinacao em execucao...';
  document.querySelectorAll('#pgPipeline .validation-step').forEach(s => s.classList.remove('running', 'pass'));
  document.getElementById('pgOutput').classList.remove('visible');
  document.getElementById('pgSummary').classList.remove('visible');

  const steps = document.querySelectorAll('#pgPipeline .validation-step');
  const stepTimes = [];
  for (const step of steps) {
    step.classList.add('running');
    const ms = 350 + Math.floor(Math.random() * 280);
    await sleep(ms);
    stepTimes.push(ms);
    step.classList.remove('running');
    step.classList.add('pass');
    const timeEl = step.querySelector('.val-step-time');
    if (timeEl) timeEl.textContent = ms + 'ms';
    await sleep(140);
  }

  await sleep(150);
  const outputEl = document.getElementById('pgOutput');
  outputEl.innerHTML = '<pre class="exec-output-body">' + scenario.output_mock + '</pre>';
  outputEl.classList.add('visible');

  await sleep(280);
  const totalMs = stepTimes.reduce((a, b) => a + b, 0);
  const tokens = 2100 + Math.floor(Math.random() * 2200);
  document.getElementById('pgSummary').innerHTML =
    '<div class="exec-summary-body">'
    + '<div><strong>' + steps.length + ' de ' + steps.length + '</strong> assertions aprovadas</div>'
    + '<div>' + (totalMs / 1000).toFixed(2) + 's · ' + tokens.toLocaleString('pt-BR') + ' tokens</div>'
    + '</div>';
  document.getElementById('pgSummary').classList.add('visible');

  runBtn.innerHTML = '<svg width="14" height="14" viewBox="0 0 24 24" fill="currentColor"><polygon points="6 4 20 12 6 20 6 4"/></svg> Executar';
  runBtn.disabled = false;
  runBtn.style.display = 'none';
  document.getElementById('pgReset').style.display = 'inline-flex';
  document.getElementById('pgGoObserv').style.display = 'inline-flex';
  document.getElementById('pgSub').innerHTML = 'Execucao concluida · <strong style="color:#22C55E">' + steps.length + '/' + steps.length + ' assertions aprovadas</strong> · ' + (totalMs/1000).toFixed(2) + 's · ' + tokens.toLocaleString('pt-BR') + ' tokens';

  emitTrace({
    id: 't' + Date.now().toString(36),
    timestamp: Date.now(),
    domain: d.id,
    domainName: d.name,
    agent: d.agents[0].role,
    scenarioId: scenario.id,
    scenarioName: scenario.name,
    status: 'pass',
    durationMs: totalMs,
    tokens,
    pipeline: Array.from(steps).map((step, i) => {
      const aName = step.dataset.assertion;
      const a = d.assertions.find(x => x.name === aName);
      return { name: aName, description: (a && a.description) || '', durationMs: stepTimes[i], status: 'pass' };
    }),
  });
}

function emitTrace(trace) {
  const list = state.ticker.recentTraces.slice();
  list.unshift(trace);
  if (list.length > 20) list.length = 20;
  setState({ ticker: Object.assign({}, state.ticker, { recentTraces: list }) });
}
```

- [ ] **Step 2: Smoke test**

ContrIA → Playground → Executar. Expected:
- Pipeline anima 4 steps com tempos variáveis
- Output mostra `scenario.output_mock` (texto plausível, não genérico)
- Sumário "4/4 aprovadas · X.XXs · N tokens"
- Console: `state.ticker.recentTraces[0].domain === 'contria'`

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat(app): playground emits real trace + real output_mock

Co-Authored-By: Claude Opus 4.7 (1M context) <noreply@anthropic.com>"
```

---

## Phase 6 — Tela Observability v2

### Task 6.1: Period sync + KPIs reativos por domínio

**Files:**
- Modify: `index.html` — `setPeriod`, `animateKpis`, tela observability

- [ ] **Step 1: Reescrever `setPeriod` para usar store**

Substituir `function setPeriod(period) { ... }` completo:

```js
function setPeriod(period) { setState({ period }); }

function renderObservability(s) {
  if (!document.getElementById('kpi1')) return;
  const d = activeDomainData();
  const m = activeMetrics();
  const periodLabel = ({'1d':'ultimas 24 horas','7d':'ultimos 7 dias','30d':'ultimos 30 dias'})[s.period];

  document.querySelectorAll('.period-tab').forEach(b => b.classList.toggle('active', b.dataset.period === s.period));

  const chartP = document.getElementById('chartPeriod');
  if (chartP) chartP.textContent = periodLabel;

  const kpi1 = document.getElementById('kpi1');
  const kpi2 = document.getElementById('kpi2');
  const kpi3 = document.getElementById('kpi3');
  const kpi4 = document.getElementById('kpi4');
  if (kpi1) animateNumber(kpi1, m.execs, 600, v => v.toLocaleString('pt-BR'));
  if (kpi2) kpi2.textContent = m.tokens;
  if (kpi3) kpi3.textContent = m.cost;
  if (kpi4) kpi4.textContent = m.approval.toFixed(1) + '%';

  const banner = document.getElementById('obDomainBanner');
  if (banner) {
    if (d) {
      banner.style.display = 'inline-flex';
      banner.querySelector('.ob-banner-name').textContent = d.fullName;
    } else {
      banner.style.display = 'none';
    }
  }

  renderTraceTable(s);
}
subscribe(renderObservability);

function animateKpis() { renderObservability(state); }
```

- [ ] **Step 2: Adicionar banner de domínio no page-head de Observability**

No HTML da tela, dentro do `<div>` que contém o `<div class="page-title">`, adicionar após o `page-sub`:

```html
<div class="ob-domain-banner" id="obDomainBanner" style="display:none">
  <span class="ob-banner-dot"></span>
  <span>Filtrado por <strong class="ob-banner-name">—</strong></span>
  <button class="ob-banner-clear" id="obBannerClear">limpar</button>
</div>
```

CSS:

```css
.ob-domain-banner {
  display: none;
  align-items: center;
  gap: 8px;
  margin-top: 8px;
  padding: 4px 12px;
  background: rgba(36, 113, 245, 0.10);
  border: 1px solid hsl(var(--primary) / 0.3);
  border-radius: 999px;
  font-size: 12px;
  color: hsl(var(--primary));
  width: fit-content;
}
.ob-banner-dot {
  width: 6px; height: 6px;
  border-radius: 50%;
  background: hsl(var(--primary));
  box-shadow: 0 0 6px hsl(var(--primary));
}
.ob-banner-clear {
  background: none; border: none;
  color: hsl(var(--primary));
  cursor: pointer;
  text-decoration: underline;
  font-size: 12px;
}
```

Listener:

```js
document.getElementById('obBannerClear').addEventListener('click', () => setState({ currentDomain: null }));
```

- [ ] **Step 3: Smoke test**

ContrIA → Playground → Executar → "Ver na Observabilidade":
- Banner "Filtrado por Contratos Administrativos"
- KPIs mostram ContrIA (4521 execs, R$ 384, etc.)
- Trocar 1d/7d/30d → KPIs reagem
- "Limpar" → banner some, KPIs voltam pro global

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat(app): observability syncs period + currentDomain via store

Co-Authored-By: Claude Opus 4.7 (1M context) <noreply@anthropic.com>"
```

### Task 6.2: Trace table conectada ao `state.ticker.recentTraces`

**Files:**
- Modify: `index.html` — tela observability, trace table

- [ ] **Step 1: Inserir HTML da trace table**

Depois dos KPIs (no `<div class="app-screen" data-screen="observability">`), adicionar:

```html
<div class="ob-traces">
  <div class="ob-traces-head">
    <h3>Execucoes recentes</h3>
    <span class="ob-traces-meta" id="obTracesMeta">—</span>
  </div>
  <table class="ob-table">
    <thead>
      <tr>
        <th>Timestamp</th><th>Dominio</th><th>Cenario</th>
        <th>Duracao</th><th>Tokens</th><th>Status</th>
      </tr>
    </thead>
    <tbody id="obTraceTbody"></tbody>
  </table>
</div>
```

CSS:

```css
.ob-traces { margin-top: 32px; }
.ob-traces-head { display: flex; justify-content: space-between; align-items: baseline; margin-bottom: 12px; }
.ob-traces-head h3 { font-family: 'Space Grotesk', sans-serif; font-size: 16px; font-weight: 600; color: #fff; }
.ob-traces-meta { font-size: 12px; color: #6B7280; }
.ob-table { width: 100%; border-collapse: collapse; font-size: 13px; }
.ob-table thead th {
  text-align: left;
  font-family: 'Space Grotesk', sans-serif;
  font-size: 11px;
  letter-spacing: 0.06em;
  text-transform: uppercase;
  color: #6B7280;
  padding: 10px 14px;
  border-bottom: 1px solid #1F2230;
  font-weight: 600;
}
.ob-table tbody tr { cursor: pointer; transition: background 0.15s; }
.ob-table tbody tr:hover { background: rgba(36, 113, 245, 0.05); }
.ob-table tbody td { padding: 12px 14px; border-bottom: 1px solid #14171F; color: #E5E7EB; }
.ob-table .trace-new { animation: traceFadeIn 0.6s ease; }
@keyframes traceFadeIn { from { background: rgba(36, 113, 245, 0.22); } to { background: transparent; } }
.ob-status-pass { display: inline-flex; align-items: center; gap: 6px; color: #22C55E; }
.ob-status-pass::before { content: ''; width: 6px; height: 6px; background: #22C55E; border-radius: 50%; }
```

- [ ] **Step 2: Implementar `renderTraceTable` + `synthInitialTraces`**

```js
function synthInitialTraces(n) {
  const out = [];
  const featured = Object.values(KAIROS_DATA.featured);
  for (let i = 0; i < n; i++) {
    const d = featured[i % featured.length];
    const sc = d.scenarios[i % d.scenarios.length];
    out.push({
      id: 't' + (Date.now() - i * 47000).toString(36) + i,
      timestamp: Date.now() - (i * 47000 + Math.random() * 30000),
      domain: d.id,
      domainName: d.name,
      agent: d.agents[0].role,
      scenarioId: sc.id,
      scenarioName: sc.name,
      status: 'pass',
      durationMs: 800 + Math.floor(Math.random() * 1100),
      tokens: 1800 + Math.floor(Math.random() * 3500),
      pipeline: d.assertions.filter(a => a.pipeline).slice(0, 4).map(a => ({
        name: a.name, description: a.description,
        durationMs: 200 + Math.floor(Math.random() * 280),
        status: 'pass',
      })),
    });
  }
  return out;
}

function renderTraceTable(s) {
  const tbody = document.getElementById('obTraceTbody');
  const meta = document.getElementById('obTracesMeta');
  if (!tbody) return;

  if (s.ticker.recentTraces.length === 0) {
    s.ticker.recentTraces = synthInitialTraces(10);
  }
  const filtered = s.currentDomain
    ? s.ticker.recentTraces.filter(t => t.domain === s.currentDomain)
    : s.ticker.recentTraces;

  meta.textContent = filtered.length + ' ' + (filtered.length === 1 ? 'trace' : 'traces') + ' · live';

  tbody.innerHTML = filtered.slice(0, 10).map(t => {
    const isNew = (Date.now() - t.timestamp) < 1500;
    const ts = new Date(t.timestamp);
    const hh = String(ts.getHours()).padStart(2, '0');
    const mm = String(ts.getMinutes()).padStart(2, '0');
    const ss = String(ts.getSeconds()).padStart(2, '0');
    return '<tr data-trace-id="' + t.id + '" class="' + (isNew ? 'trace-new' : '') + '">'
      + '<td><code>' + hh + ':' + mm + ':' + ss + '</code></td>'
      + '<td><strong>' + t.domainName + '</strong></td>'
      + '<td>' + t.scenarioName + '</td>'
      + '<td><code>' + (t.durationMs/1000).toFixed(2) + 's</code></td>'
      + '<td><code>' + t.tokens.toLocaleString('pt-BR') + '</code></td>'
      + '<td><span class="ob-status-pass">' + t.status + '</span></td>'
      + '</tr>';
  }).join('');

  tbody.querySelectorAll('tr').forEach(tr => {
    tr.addEventListener('click', () => {
      const trace = s.ticker.recentTraces.find(t => t.id === tr.dataset.traceId);
      if (trace) openDrill('trace', trace);
    });
  });
}
```

- [ ] **Step 3: Substituir stub `renderTraceDrill`**

```js
function renderTraceDrill(t, eyebrow, title, body, foot) {
  eyebrow.textContent = 'Execucao';
  title.textContent = t.domainName + ' · ' + t.scenarioName;
  const ts = new Date(t.timestamp);
  body.innerHTML =
    '<div class="drill-section">'
    + '<dl class="drill-kv">'
    + '<dt>ID</dt><dd><code>' + t.id + '</code></dd>'
    + '<dt>Quando</dt><dd>' + ts.toLocaleString('pt-BR') + '</dd>'
    + '<dt>Agente</dt><dd>' + t.agent + '</dd>'
    + '<dt>Duracao</dt><dd>' + (t.durationMs/1000).toFixed(2) + 's</dd>'
    + '<dt>Tokens</dt><dd>' + t.tokens.toLocaleString('pt-BR') + '</dd>'
    + '<dt>Status</dt><dd><span class="drill-tag success">' + t.status + '</span></dd>'
    + '</dl>'
    + '</div>'
    + '<div class="drill-section">'
    + '<div class="drill-section-title">Pipeline anti-alucinacao</div>'
    + '<div class="drill-list">'
    + t.pipeline.map((p, i) =>
      '<div class="drill-list-item">'
      + '<div class="di-title">' + (i + 1) + '. ' + p.description + '</div>'
      + '<div class="di-meta"><code>' + p.name + '</code> · ' + p.durationMs + 'ms · <span class="drill-tag success" style="margin-left:4px">' + p.status + '</span></div>'
      + '</div>'
    ).join('')
    + '</div>'
    + '</div>';
  foot.innerHTML = '<button class="app-btn secondary" id="drillCancelBtn">Fechar</button>';
  document.getElementById('drillCancelBtn').addEventListener('click', closeDrill);
}
```

- [ ] **Step 4: Smoke test**

Observability:
- Tabela com 10 traces sintéticas (mix dos 3 domínios)
- Clicar linha → drill com pipeline expandido
- Playground → Executar → "Ver na Observabilidade" → trace nova no topo com animação
- Selecionar ContrIA → só linhas de ContrIA

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat(app): observability trace table from store + per-trace drill

Co-Authored-By: Claude Opus 4.7 (1M context) <noreply@anthropic.com>"
```

---

## Phase 7 — Tela FinOps v2

### Task 7.1: Cost-by-model reativo

**Files:**
- Modify: `index.html` — tela finops

- [ ] **Step 1: Substituir conteúdo da lista de modelos**

Localizar `<div class="fo-models-list">` e substituir conteúdo por:

```html
<div class="fo-models-list" id="foModelsList"></div>
```

- [ ] **Step 2: Adicionar `renderFinOps`**

```js
// FINOPS
function formatCurrency(v) {
  return 'US$ ' + v.toLocaleString('en-US', { minimumFractionDigits: 2, maximumFractionDigits: 2 });
}

function renderFinOps(s) {
  const list = document.getElementById('foModelsList');
  if (!list) return;

  const d = activeDomainData();
  const finops = d ? d.finops : KAIROS_DATA.global.finops;
  const multiplier = ({'1d': 1/7, '7d': 1, '30d': 4.3})[s.period] || 1;

  list.innerHTML = finops.models.map((m, i) => {
    const cost = m.cost * multiplier;
    const gradient = i > 0 ? 'background: linear-gradient(90deg, hsl(var(--secondary)), hsl(var(--accent)));' : '';
    return '<div class="fo-model" data-model-name="' + m.name + '">'
      + '<div class="fo-model-info">'
      + '<div class="fo-model-name">' + m.name + '</div>'
      + '<div class="fo-model-bar-track"><div class="fo-model-bar-fill" style="width:' + m.pct + '%;' + gradient + '"></div></div>'
      + '</div>'
      + '<div>'
      + '<div class="fo-model-cost">' + formatCurrency(cost) + '</div>'
      + '<div class="fo-model-pct">' + m.pct + '% · ' + m.tokens + '</div>'
      + '</div>'
      + '</div>';
  }).join('');

  list.querySelectorAll('.fo-model').forEach(el => {
    el.addEventListener('click', () => {
      const name = el.dataset.modelName;
      const m = finops.models.find(mm => mm.name === name);
      if (m) openDrill('model', Object.assign({}, m, {
        domainName: d ? d.fullName : 'Agregado',
        topPrompts: ((d && d.finops && d.finops.topPrompts) || []).filter(p => p.model === name),
      }));
    });
  });

  renderFinOpsBudget(s);
}
subscribe(renderFinOps);
```

- [ ] **Step 3: Smoke test**

Console:

```js
setState({ currentDomain: 'contria' })  // → vê 3 modelos
setState({ currentDomain: null })       // → vê 4 modelos globais
```

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat(app): finops cost-by-model reactive to period+domain

Co-Authored-By: Claude Opus 4.7 (1M context) <noreply@anthropic.com>"
```

### Task 7.2: Budget alert pulsante + drills

**Files:**
- Modify: `index.html` — adicionar card de budget + renderers

- [ ] **Step 1: Inserir HTML do card**

Em `data-screen="finops"`, após `fo-models-card`:

```html
<div class="fo-budget-card" id="foBudgetCard">
  <div class="fo-budget-head">
    <div>
      <div class="fo-budget-eyebrow">Orcamento mensal</div>
      <div class="fo-budget-title" id="foBudgetTitle">—</div>
    </div>
    <span class="fo-budget-pulse" id="foBudgetPulse"></span>
  </div>
  <div class="fo-budget-bar-track">
    <div class="fo-budget-bar-fill" id="foBudgetFill" style="width:0%"></div>
  </div>
  <div class="fo-budget-meta" id="foBudgetMeta">—</div>
</div>
```

CSS:

```css
.fo-budget-card {
  background: #0B0C12;
  border: 1px solid #1F2230;
  border-radius: 14px;
  padding: 20px 22px;
  margin-top: 20px;
  cursor: pointer;
  transition: border-color 0.18s, box-shadow 0.18s;
}
.fo-budget-card:hover { border-color: hsl(var(--primary) / 0.4); box-shadow: 0 8px 22px hsl(var(--primary) / 0.12); }
.fo-budget-head { display: flex; justify-content: space-between; align-items: flex-start; margin-bottom: 14px; }
.fo-budget-eyebrow { font-size: 11px; letter-spacing: 0.08em; text-transform: uppercase; color: #6B7280; font-weight: 600; margin-bottom: 6px; }
.fo-budget-title { font-family: 'Space Grotesk', sans-serif; font-size: 18px; font-weight: 700; color: #fff; }
.fo-budget-pulse {
  width: 12px; height: 12px; border-radius: 50%;
  background: #F59E0B;
  box-shadow: 0 0 0 0 rgba(245, 158, 11, 0.55);
  animation: budgetPulse 2.4s ease-out infinite;
}
.fo-budget-card.danger .fo-budget-pulse { background: #EF4444; box-shadow: 0 0 0 0 rgba(239, 68, 68, 0.55); }
@keyframes budgetPulse {
  0%   { box-shadow: 0 0 0 0 rgba(245, 158, 11, 0.55); }
  70%  { box-shadow: 0 0 0 12px rgba(245, 158, 11, 0); }
  100% { box-shadow: 0 0 0 0 rgba(245, 158, 11, 0); }
}
.fo-budget-bar-track { height: 10px; background: rgba(255, 255, 255, 0.04); border-radius: 5px; overflow: hidden; margin-bottom: 10px; }
.fo-budget-bar-fill {
  height: 100%;
  background: linear-gradient(90deg, #F59E0B, #EF4444);
  border-radius: 5px;
  transition: width 0.5s ease;
}
.fo-budget-meta { font-size: 13px; color: #9CA3AF; }
```

- [ ] **Step 2: `renderFinOpsBudget` + drills substituídos**

```js
function renderFinOpsBudget(s) {
  const card = document.getElementById('foBudgetCard');
  if (!card) return;
  const d = activeDomainData();
  const finops = d ? d.finops : KAIROS_DATA.global.finops;
  const b = finops.budget;
  document.getElementById('foBudgetTitle').textContent = (d ? d.name : 'Sistema agregado') + ' · ' + b.used + '% do orcamento';
  document.getElementById('foBudgetFill').style.width = b.used + '%';
  document.getElementById('foBudgetMeta').textContent =
    b.currency + ' ' + b.current.toLocaleString('pt-BR') + ' de ' + b.currency + ' ' + b.limit.toLocaleString('pt-BR') + ' consumidos no mes — projecao EOM: ' + b.currency + ' ' + Math.round(b.current * 1.18).toLocaleString('pt-BR');
  card.classList.toggle('danger', b.used >= 80);
  card.onclick = () => openDrill('budget', Object.assign({}, b, { domainName: d ? d.fullName : 'Sistema agregado' }));
}

function renderBudgetDrill(b, eyebrow, title, body, foot) {
  eyebrow.textContent = 'Orcamento';
  title.textContent = b.domainName;
  body.innerHTML =
    '<div class="drill-section">'
    + '<dl class="drill-kv">'
    + '<dt>Limite mensal</dt><dd>' + b.currency + ' ' + b.limit.toLocaleString('pt-BR') + '</dd>'
    + '<dt>Consumido</dt><dd>' + b.currency + ' ' + b.current.toLocaleString('pt-BR') + ' (' + b.used + '%)</dd>'
    + '<dt>Restante</dt><dd>' + b.currency + ' ' + (b.limit - b.current).toLocaleString('pt-BR') + '</dd>'
    + '<dt>Projecao EOM</dt><dd>' + b.currency + ' ' + Math.round(b.current * 1.18).toLocaleString('pt-BR') + '</dd>'
    + '</dl></div>'
    + '<div class="drill-section">'
    + '<div class="drill-section-title">Historico (ultimos 30 dias)</div>'
    + '<p>Consumo medio diario: ' + b.currency + ' ' + Math.round(b.current / 22).toLocaleString('pt-BR') + '. Pico em D-3 (' + b.currency + ' ' + Math.round(b.current / 16).toLocaleString('pt-BR') + ') por execucao em lote do squad fiscal-contrato.</p>'
    + '</div>'
    + '<div class="drill-section">'
    + '<div class="drill-section-title">Recomendacao</div>'
    + '<p>' + (b.used >= 80 ? 'Considerar elevacao do teto ou cap por agente fiscal.' : 'Sob controle. Sem acao requerida.') + '</p>'
    + '</div>';
  foot.innerHTML = '<button class="app-btn secondary" id="drillCancelBtn">Fechar</button>';
  document.getElementById('drillCancelBtn').addEventListener('click', closeDrill);
}

function renderModelDrill(m, eyebrow, title, body, foot) {
  eyebrow.textContent = 'Modelo · ' + m.domainName;
  title.textContent = m.name;
  const topHtml = (m.topPrompts || []).length === 0
    ? '<p style="color:#6B7280">Sem detalhamento por prompt para este modelo no recorte atual.</p>'
    : m.topPrompts.map(p =>
        '<div class="drill-list-item">'
        + '<div class="di-title">' + p.label + '</div>'
        + '<div class="di-meta">' + p.runs + ' execucoes · ' + formatCurrency(p.costSum) + '</div>'
        + '</div>'
      ).join('');
  body.innerHTML =
    '<div class="drill-section">'
    + '<dl class="drill-kv">'
    + '<dt>Custo (periodo)</dt><dd>' + formatCurrency(m.cost) + '</dd>'
    + '<dt>Tokens</dt><dd>' + m.tokens + '</dd>'
    + '<dt>Share</dt><dd>' + m.pct + '%</dd>'
    + '</dl></div>'
    + '<div class="drill-section">'
    + '<div class="drill-section-title">Top prompts por custo</div>'
    + '<div class="drill-list">' + topHtml + '</div>'
    + '</div>';
  foot.innerHTML = '<button class="app-btn secondary" id="drillCancelBtn">Fechar</button>';
  document.getElementById('drillCancelBtn').addEventListener('click', closeDrill);
}
```

- [ ] **Step 3: Smoke test**

FinOps:
- Card de budget aparece, dot laranja pulsando
- "Sistema agregado · 64%" sem domínio
- Selecionar ContrIA → "ContrIA · 78%", barra mais cheia, dot continua laranja
- Clicar card → drill com histórico
- Clicar modelo → drill com top prompts

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat(app): finops budget card pulsing + drills (model, budget)

Co-Authored-By: Claude Opus 4.7 (1M context) <noreply@anthropic.com>"
```

### Task 7.3: Period tabs em FinOps

**Files:**
- Modify: `index.html` — adicionar tabs no page-head de FinOps

- [ ] **Step 1: Inserir period tabs**

No `<div class="page-actions">` do FinOps (criar se ausente):

```html
<div class="period-tabs" data-period-group="finops">
  <button class="period-tab" data-period="1d">1d</button>
  <button class="period-tab active" data-period="7d">7d</button>
  <button class="period-tab" data-period="30d">30d</button>
</div>
```

- [ ] **Step 2: Generalizar listener**

Substituir o listener antigo `document.querySelectorAll('#periodTabs .period-tab')...` por:

```js
document.querySelectorAll('.period-tabs .period-tab').forEach(btn => {
  btn.addEventListener('click', () => setPeriod(btn.dataset.period));
});
```

- [ ] **Step 3: Smoke test**

FinOps → trocar 1d/7d/30d. Custos escalam (1/7×, 1×, 4.3×). Budget bar não muda (mensal). KPIs em Observability acompanham.

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat(app): finops period tabs synced with state.period

Co-Authored-By: Claude Opus 4.7 (1M context) <noreply@anthropic.com>"
```

---

## Phase 8 — Tela Compliance v2

### Task 8.1: Categorias + items funcionais

**Files:**
- Modify: `index.html` — tela compliance

- [ ] **Step 1: Limpar items hardcoded**

Substituir a lista de items existente por:

```html
<div class="co-items" id="coItemsList"></div>
```

Garantir que cada `.co-cat` tem `data-cat="lei14133|lgpd|lai|acessibilidade"`. Adicionar se ausente.

- [ ] **Step 2: Adicionar `renderCompliance`**

```js
// COMPLIANCE
function prettyCategoryName(cat) {
  return ({ lei14133: 'Lei 14.133/21', lgpd: 'LGPD', lai: 'LAI', acessibilidade: 'Acessibilidade' })[cat] || cat;
}

function renderCompliance(s) {
  const list = document.getElementById('coItemsList');
  if (!list) return;

  const cat = s.filters.compliance.category;
  const d = activeDomainData();

  document.querySelectorAll('#kairosApp .co-cat').forEach(c => {
    c.classList.toggle('active', c.dataset.cat === cat);
    c.classList.toggle('domain-applicable', !!(d && d.compliance.categories.includes(c.dataset.cat)));
  });

  let items;
  if (d && d.compliance.categories.includes(cat)) {
    items = d.compliance.items;
  } else if (!d && cat === 'lei14133') {
    items = KAIROS_DATA.featured.contria.compliance.items.concat(KAIROS_DATA.featured.licitia.compliance.items);
  } else if (!d && cat === 'lgpd') {
    items = KAIROS_DATA.featured.privacia.compliance.items;
  } else if (!d && cat === 'lai') {
    items = [
      { label: 'Resposta a pedido LAI <= 20 dias uteis', regulator: 'Lei 12.527/11', lastCheck: 'pass', passRate7d: 0.97 },
      { label: 'Transparencia ativa atualizada', regulator: 'Decreto 7.724/12', lastCheck: 'pass', passRate7d: 0.99 },
    ];
  } else if (!d && cat === 'acessibilidade') {
    items = [
      { label: 'Documentos PDF com OCR e texto selecionavel', regulator: 'LBI · Decreto 9.508/18', lastCheck: 'warning', passRate7d: 0.82 },
      { label: 'Contraste AA conforme WCAG 2.1', regulator: 'eMAG', lastCheck: 'pass', passRate7d: 0.95 },
    ];
  } else {
    items = [];
  }

  if (items.length === 0) {
    list.innerHTML =
      '<div class="co-empty">'
      + '<div>Categoria <strong>' + prettyCategoryName(cat) + '</strong> nao e aplicavel a <strong>' + (d ? d.fullName : 'sistema') + '</strong>.</div>'
      + '<button class="app-btn secondary" id="coClearDomain">Ver agregado do sistema</button>'
      + '</div>';
    document.getElementById('coClearDomain').addEventListener('click', () => setState({ currentDomain: null }));
    return;
  }

  list.innerHTML = items.map((it, i) => {
    const tagCls = it.lastCheck === 'pass' ? 'success' : it.lastCheck === 'warning' ? 'warn' : 'fail';
    return '<article class="co-item" data-item-i="' + i + '">'
      + '<div class="co-item-head">'
      + '<div class="co-item-label">' + it.label + '</div>'
      + '<span class="drill-tag ' + tagCls + '">' + it.lastCheck + '</span>'
      + '</div>'
      + '<div class="co-item-meta">' + it.regulator + ' · taxa de aprovacao 7d: <strong>' + (it.passRate7d * 100).toFixed(1) + '%</strong></div>'
      + '</article>';
  }).join('');

  list.querySelectorAll('.co-item').forEach(el => {
    el.addEventListener('click', () => openDrill('compliance-item', items[parseInt(el.dataset.itemI, 10)]));
  });
}
subscribe(renderCompliance);

document.querySelectorAll('#kairosApp .co-cat').forEach(catEl => {
  catEl.addEventListener('click', () => {
    setState({ filters: Object.assign({}, state.filters, { compliance: { category: catEl.dataset.cat } }) });
  });
});
```

- [ ] **Step 3: CSS empty + domain-applicable + co-items**

```css
.co-cat.domain-applicable { position: relative; }
.co-cat.domain-applicable::after {
  content: 'aplicavel';
  position: absolute;
  top: 4px; right: 4px;
  font-size: 9px;
  letter-spacing: 0.06em;
  text-transform: uppercase;
  background: hsl(var(--primary) / 0.15);
  color: hsl(var(--primary));
  padding: 2px 6px;
  border-radius: 999px;
  font-weight: 600;
}
.co-items { display: grid; grid-template-columns: 1fr; gap: 10px; margin-top: 18px; }
.co-item {
  background: #0B0C12;
  border: 1px solid #1F2230;
  border-radius: 12px;
  padding: 16px 20px;
  cursor: pointer;
  transition: border-color 0.18s, transform 0.18s;
}
.co-item:hover { border-color: hsl(var(--primary) / 0.4); transform: translateX(2px); }
.co-item-head { display: flex; justify-content: space-between; align-items: center; margin-bottom: 6px; gap: 12px; }
.co-item-label { font-weight: 600; color: #fff; font-size: 14px; }
.co-item-meta { font-size: 12px; color: #9CA3AF; }
.co-empty {
  padding: 28px;
  background: rgba(255, 255, 255, 0.02);
  border: 1px dashed #1F2230;
  border-radius: 12px;
  display: flex; flex-direction: column; gap: 14px;
  align-items: center;
  color: #9CA3AF; font-size: 14px;
  text-align: center;
}
```

- [ ] **Step 4: Substituir stub `renderComplianceItemDrill`**

```js
function renderComplianceItemDrill(c, eyebrow, title, body, foot) {
  const tagCls = c.lastCheck === 'pass' ? 'success' : c.lastCheck === 'warning' ? 'warn' : 'fail';
  eyebrow.textContent = 'Item de compliance';
  title.textContent = c.label;
  body.innerHTML =
    '<div class="drill-section">'
    + '<dl class="drill-kv">'
    + '<dt>Regulacao</dt><dd>' + c.regulator + '</dd>'
    + '<dt>Ultima checagem</dt><dd><span class="drill-tag ' + tagCls + '">' + c.lastCheck + '</span></dd>'
    + '<dt>Taxa 7d</dt><dd>' + (c.passRate7d * 100).toFixed(1) + '%</dd>'
    + '</dl></div>'
    + '<div class="drill-section">'
    + '<div class="drill-section-title">Como e verificado</div>'
    + '<p>Esta assertion roda em todas as execucoes dos agentes cobertos pelo dominio. Falhas geram alerta ao responsavel e bloqueio de saida em campos sensiveis (fundamentacao, citacao legal).</p>'
    + '</div>';
  foot.innerHTML = '<button class="app-btn secondary" id="drillCancelBtn">Fechar</button>';
  document.getElementById('drillCancelBtn').addEventListener('click', closeDrill);
}
```

- [ ] **Step 5: Smoke test**

Compliance:
- 4 categorias, Lei 14.133 ativa
- Items para Lei 14.133 (mescla ContrIA + LicitIA)
- ContrIA via Catálogo → Lei 14.133 com badge "aplicável", items só de ContrIA
- Clicar item → drill
- PrivacIA + Lei 14.133 → estado empty com botão fallback

- [ ] **Step 6: Commit**

```bash
git add index.html
git commit -m "feat(app): compliance v2 - categories functional + per-domain items + drill

Co-Authored-By: Claude Opus 4.7 (1M context) <noreply@anthropic.com>"
```

---

## Phase 9 — Telas Agents / Assertions

### Task 9.1: Agents — lista, chips, drill

**Files:**
- Modify: `index.html` — tela agents

- [ ] **Step 1: Substituir lista por container**

Localizar a lista estática em `data-screen="agents"`. Substituir por:

```html
<div class="ag-list" id="agentsList"></div>
```

Confirmar que chips têm `data-role="all|negocial|fiscal|tecnico|auditor"` ou ajustar.

- [ ] **Step 2: `renderAgents` + listeners**

```js
// AGENTS
function renderAgents(s) {
  const list = document.getElementById('agentsList');
  if (!list) return;

  const d = activeDomainData();
  const all = d
    ? d.agents.map(a => Object.assign({}, a, { _domain: d }))
    : Object.values(KAIROS_DATA.featured).flatMap(dd => dd.agents.map(a => Object.assign({}, a, { _domain: dd })));

  const roleFilter = s.filters.agents.role;
  const filtered = roleFilter === 'all'
    ? all
    : all.filter(a => (a.role + ' ' + (a.specialty || '')).toLowerCase().includes(roleFilter.toLowerCase()));

  list.innerHTML = filtered.length === 0
    ? '<div class="ag-empty">Nenhum agente bate com o filtro.</div>'
    : filtered.map((a, i) =>
        '<article class="ag-row" data-i="' + i + '">'
        + '<div class="ag-row-main">'
        + '<div class="ag-row-title">' + a.role + '</div>'
        + '<div class="ag-row-sub">' + (a.specialty || a.description || '') + '</div>'
        + '</div>'
        + '<div class="ag-row-meta"><span class="drill-tag">' + a._domain.name + '</span></div>'
        + '</article>'
      ).join('');

  list.querySelectorAll('.ag-row').forEach(el => {
    el.addEventListener('click', () => openDrill('agent', filtered[parseInt(el.dataset.i, 10)]));
  });
}
subscribe(renderAgents);

document.querySelectorAll('#kairosApp [data-screen="agents"] .ag-chip').forEach(chip => {
  chip.addEventListener('click', () => {
    document.querySelectorAll('#kairosApp [data-screen="agents"] .ag-chip').forEach(c => c.classList.remove('active'));
    chip.classList.add('active');
    setState({ filters: Object.assign({}, state.filters, { agents: { role: chip.dataset.role || 'all' } }) });
  });
});
```

- [ ] **Step 3: CSS lista de agents**

```css
.ag-list { display: grid; grid-template-columns: 1fr; gap: 10px; margin-top: 18px; }
.ag-row {
  display: flex; justify-content: space-between; align-items: center; gap: 14px;
  padding: 14px 18px;
  background: #0B0C12;
  border: 1px solid #1F2230;
  border-radius: 10px;
  cursor: pointer;
  transition: all 0.15s;
}
.ag-row:hover { border-color: hsl(var(--primary) / 0.4); background: rgba(36, 113, 245, 0.04); }
.ag-row-title { font-weight: 600; color: #fff; margin-bottom: 3px; font-size: 14px; }
.ag-row-sub { font-size: 12px; color: #9CA3AF; line-height: 1.5; }
.ag-row-meta { flex-shrink: 0; }
.ag-empty { padding: 32px; color: #6B7280; text-align: center; font-size: 13px; }
```

- [ ] **Step 4: Substituir stub `renderAgentDrill`**

```js
function renderAgentDrill(a, eyebrow, title, body, foot) {
  eyebrow.textContent = 'Agente · ' + a._domain.name;
  title.textContent = a.role;
  body.innerHTML =
    '<div class="drill-section">'
    + '<dl class="drill-kv">'
    + '<dt>ID</dt><dd><code>' + (a.id || a.role.toLowerCase().replace(/\s+/g, '-')) + '</code></dd>'
    + '<dt>Dominio</dt><dd>' + a._domain.fullName + '</dd>'
    + '<dt>Especialidade</dt><dd>' + (a.specialty || '—') + '</dd>'
    + '</dl></div>'
    + '<div class="drill-section">'
    + '<div class="drill-section-title">Descricao</div>'
    + '<p>' + (a.description || '—') + '</p>'
    + '</div>'
    + '<div class="drill-section">'
    + '<div class="drill-section-title">Assertions vinculadas (top 5)</div>'
    + '<div class="drill-list">'
    + a._domain.assertions.slice(0, 5).map(as =>
        '<div class="drill-list-item">'
        + '<div class="di-title"><code>' + as.name + '</code></div>'
        + '<div class="di-meta">' + as.description + '</div>'
        + '</div>'
      ).join('')
    + '</div></div>';
  foot.innerHTML = '<button class="app-btn secondary" id="drillCancelBtn">Fechar</button>';
  document.getElementById('drillCancelBtn').addEventListener('click', closeDrill);
}
```

- [ ] **Step 5: Smoke test**

Agents:
- Lista das 3 GovAI featured
- ContrIA via Catálogo → só ContrIA
- Chip filtra
- Row click → drill

- [ ] **Step 6: Commit**

```bash
git add index.html
git commit -m "feat(app): agents v2 - list/filter/drill from KAIROS_DATA

Co-Authored-By: Claude Opus 4.7 (1M context) <noreply@anthropic.com>"
```

### Task 9.2: Assertions — lista, chips, drill

**Files:**
- Modify: `index.html` — tela assertions

- [ ] **Step 1: Substituir lista**

```html
<div class="as-list" id="assertionsList"></div>
```

Chips: `data-status="all|pass|warning|fail"`.

- [ ] **Step 2: `renderAssertions` + listeners**

```js
// ASSERTIONS
function renderAssertions(s) {
  const list = document.getElementById('assertionsList');
  if (!list) return;

  const d = activeDomainData();
  const all = d
    ? d.assertions.map(a => Object.assign({}, a, { _domain: d }))
    : Object.values(KAIROS_DATA.featured).flatMap(dd => dd.assertions.map(a => Object.assign({}, a, { _domain: dd })));

  function statusFor(a) {
    const h = a.name.split('').reduce((acc, c) => acc + c.charCodeAt(0), 0);
    if (h % 13 === 0) return 'fail';
    if (h % 7 === 0) return 'warning';
    return 'pass';
  }
  const enriched = all.map(a => Object.assign({}, a, { status: statusFor(a) }));

  const sf = s.filters.assertions.status;
  const filtered = sf === 'all' ? enriched : enriched.filter(a => a.status === sf);

  list.innerHTML = filtered.length === 0
    ? '<div class="ag-empty">Nenhuma assertion bate com o filtro.</div>'
    : filtered.map((a, i) => {
        const tagCls = a.status === 'pass' ? 'success' : a.status === 'warning' ? 'warn' : 'fail';
        return '<article class="as-row" data-i="' + i + '">'
          + '<div class="as-row-main">'
          + '<code class="as-row-name">' + a.name + '</code>'
          + '<div class="as-row-sub">' + a.description + '</div>'
          + '</div>'
          + '<div class="as-row-meta">'
          + '<span class="drill-tag ' + tagCls + '">' + a.status + '</span>'
          + '<span class="drill-tag">' + a._domain.name + '</span>'
          + '</div>'
          + '</article>';
      }).join('');

  list.querySelectorAll('.as-row').forEach(el => {
    el.addEventListener('click', () => openDrill('assertion', filtered[parseInt(el.dataset.i, 10)]));
  });
}
subscribe(renderAssertions);

document.querySelectorAll('#kairosApp [data-screen="assertions"] .ag-chip').forEach(chip => {
  chip.addEventListener('click', () => {
    document.querySelectorAll('#kairosApp [data-screen="assertions"] .ag-chip').forEach(c => c.classList.remove('active'));
    chip.classList.add('active');
    setState({ filters: Object.assign({}, state.filters, { assertions: { status: chip.dataset.status || 'all' } }) });
  });
});
```

- [ ] **Step 3: CSS lista de assertions**

```css
.as-list { display: grid; grid-template-columns: 1fr; gap: 10px; margin-top: 18px; }
.as-row {
  display: flex; justify-content: space-between; align-items: center; gap: 14px;
  padding: 14px 18px;
  background: #0B0C12;
  border: 1px solid #1F2230;
  border-radius: 10px;
  cursor: pointer;
  transition: all 0.15s;
}
.as-row:hover { border-color: hsl(var(--primary) / 0.4); background: rgba(36, 113, 245, 0.04); }
.as-row-name { font-family: 'JetBrains Mono', monospace; font-size: 12px; color: hsl(var(--primary)); font-weight: 500; margin-bottom: 4px; display: block; }
.as-row-sub { font-size: 13px; color: #E5E7EB; line-height: 1.5; }
.as-row-meta { flex-shrink: 0; display: flex; flex-direction: column; gap: 4px; align-items: flex-end; }
```

- [ ] **Step 4: Substituir stub `renderAssertionDrill`**

```js
function renderAssertionDrill(a, eyebrow, title, body, foot) {
  const tagCls = a.status === 'pass' ? 'success' : a.status === 'warning' ? 'warn' : 'fail';
  eyebrow.textContent = 'Assertion · ' + a._domain.name;
  title.innerHTML = '<code style="font-size:18px">' + a.name + '</code>';
  const passRate7d = a.status === 'pass'
    ? (0.95 + Math.random() * 0.05)
    : a.status === 'warning' ? (0.80 + Math.random() * 0.10) : (0.60 + Math.random() * 0.15);
  const scenariosHtml = a._domain.scenarios.filter(sc => (sc.expected_keys || []).includes(a.name)).map(sc =>
    '<div class="drill-list-item">'
    + '<div class="di-title">' + sc.name + '</div>'
    + '<div class="di-meta">' + sc.input.slice(0, 120) + (sc.input.length > 120 ? '…' : '') + '</div>'
    + '</div>'
  ).join('') || '<p style="color:#6B7280">Esta assertion nao e exercitada diretamente pelos cenarios curados; e validada inline em todas as execucoes.</p>';
  body.innerHTML =
    '<div class="drill-section">'
    + '<dl class="drill-kv">'
    + '<dt>Dominio</dt><dd>' + a._domain.fullName + '</dd>'
    + '<dt>Categoria</dt><dd>' + (a.category || '—') + '</dd>'
    + '<dt>Pipeline</dt><dd>' + (a.pipeline ? '<span class="drill-tag success">Sim · ordem ' + a.pipelineOrder + '</span>' : '<span class="drill-tag">Nao</span>') + '</dd>'
    + '<dt>Status atual</dt><dd><span class="drill-tag ' + tagCls + '">' + a.status + '</span></dd>'
    + '<dt>Taxa 7d</dt><dd>' + (passRate7d * 100).toFixed(1) + '%</dd>'
    + '</dl></div>'
    + '<div class="drill-section">'
    + '<div class="drill-section-title">Descricao</div>'
    + '<p>' + a.description + '</p>'
    + '</div>'
    + '<div class="drill-section">'
    + '<div class="drill-section-title">Cenarios que exercitam</div>'
    + '<div class="drill-list">' + scenariosHtml + '</div>'
    + '</div>';
  foot.innerHTML = '<button class="app-btn secondary" id="drillCancelBtn">Fechar</button>';
  document.getElementById('drillCancelBtn').addEventListener('click', closeDrill);
}
```

- [ ] **Step 5: Smoke test**

Assertions:
- Lista das 3 GovAI
- Filtro por status muda
- Drill mostra scenarios linkados

- [ ] **Step 6: Commit**

```bash
git add index.html
git commit -m "feat(app): assertions v2 - list/filter/drill from KAIROS_DATA

Co-Authored-By: Claude Opus 4.7 (1M context) <noreply@anthropic.com>"
```

---

## Phase 10 — Tela Overview v2

### Task 10.1: Hero KPIs + quick cards + activity feed

**Files:**
- Modify: `index.html` — tela overview

- [ ] **Step 1: Substituir markup da Overview**

Trocar todo conteúdo de `<div class="app-screen" data-screen="overview">` por:

```html
<div class="app-screen" data-screen="overview">
  <div class="studio-main">
    <div class="page-head">
      <div>
        <div class="page-title">Visao geral · KairOS Studio</div>
        <div class="page-sub">Tenant <strong>vilelaai-prod</strong> · periodo <span id="ovPeriodLabel">ultimos 7 dias</span></div>
      </div>
      <div class="page-actions">
        <div class="period-tabs" data-period-group="overview">
          <button class="period-tab" data-period="1d">1d</button>
          <button class="period-tab active" data-period="7d">7d</button>
          <button class="period-tab" data-period="30d">30d</button>
        </div>
      </div>
    </div>

    <div class="ov-banner" id="ovDomainBanner" style="display:none">
      <span class="ob-banner-dot"></span>
      <span>Vendo dados de <strong class="ov-banner-name">—</strong></span>
      <button class="ob-banner-clear" id="ovBannerClear">limpar</button>
    </div>

    <div class="ov-hero-kpis">
      <div class="ov-kpi"><div class="ov-kpi-label">Execucoes</div><div class="ov-kpi-value" id="ovKpiExecs">—</div></div>
      <div class="ov-kpi"><div class="ov-kpi-label">Tokens</div><div class="ov-kpi-value" id="ovKpiTokens">—</div></div>
      <div class="ov-kpi"><div class="ov-kpi-label">Custo</div><div class="ov-kpi-value" id="ovKpiCost">—</div></div>
      <div class="ov-kpi"><div class="ov-kpi-label">Aprovacao</div><div class="ov-kpi-value" id="ovKpiApproval">—</div></div>
    </div>

    <div class="ov-quick-cards" id="ovQuickCards"></div>

    <div class="ov-feed">
      <div class="ov-feed-head">
        <h3>Atividade recente</h3>
        <span class="ob-traces-meta">live</span>
      </div>
      <div class="ov-feed-list" id="ovFeedList"></div>
    </div>
  </div>
</div>
```

- [ ] **Step 2: CSS Overview**

```css
.ov-hero-kpis { display: grid; grid-template-columns: repeat(4, 1fr); gap: 12px; margin-top: 24px; }
.ov-kpi {
  background: #0B0C12;
  border: 1px solid #1F2230;
  border-radius: 14px;
  padding: 20px 22px;
}
.ov-kpi-label { font-size: 11px; letter-spacing: 0.08em; text-transform: uppercase; color: #6B7280; font-weight: 600; margin-bottom: 10px; }
.ov-kpi-value { font-family: 'Space Grotesk', sans-serif; font-size: 32px; font-weight: 700; color: #fff; font-variant-numeric: tabular-nums; }

.ov-quick-cards { display: grid; grid-template-columns: repeat(3, 1fr); gap: 12px; margin-top: 28px; }
.ov-quick-card {
  background: linear-gradient(135deg, rgba(36, 113, 245, 0.08), rgba(107, 38, 214, 0.08));
  border: 1px solid hsl(var(--primary) / 0.3);
  border-radius: 14px;
  padding: 20px 22px;
  cursor: pointer;
  transition: transform 0.18s, box-shadow 0.18s, border-color 0.18s;
}
.ov-quick-card:hover { transform: translateY(-3px); border-color: hsl(var(--primary) / 0.6); box-shadow: 0 12px 30px hsl(var(--primary) / 0.18); }
.ov-quick-card-eyebrow { font-size: 11px; letter-spacing: 0.08em; text-transform: uppercase; color: hsl(var(--primary)); font-weight: 600; margin-bottom: 6px; }
.ov-quick-card-title { font-family: 'Space Grotesk', sans-serif; font-size: 20px; font-weight: 700; color: #fff; margin-bottom: 8px; }
.ov-quick-card-meta { font-size: 12px; color: #9CA3AF; }

.ov-feed { margin-top: 32px; }
.ov-feed-head { display: flex; justify-content: space-between; align-items: baseline; margin-bottom: 12px; }
.ov-feed-head h3 { font-family: 'Space Grotesk', sans-serif; font-size: 16px; font-weight: 600; color: #fff; }
.ov-feed-list { display: grid; grid-template-columns: 1fr; gap: 8px; }
.ov-feed-row {
  display: grid;
  grid-template-columns: 80px 1fr auto auto;
  gap: 14px;
  align-items: center;
  padding: 10px 16px;
  background: #0B0C12;
  border: 1px solid #1F2230;
  border-radius: 8px;
  cursor: pointer;
  transition: background 0.15s;
}
.ov-feed-row:hover { background: rgba(36, 113, 245, 0.04); }
.ov-feed-time { font-family: 'JetBrains Mono', monospace; font-size: 12px; color: #6B7280; }
.ov-feed-title { color: #fff; font-size: 13px; }
.ov-feed-title strong { font-weight: 600; }
.ov-feed-meta { font-size: 12px; color: #9CA3AF; }

.ov-banner {
  margin-top: 14px;
  display: inline-flex; align-items: center; gap: 8px;
  padding: 4px 12px;
  background: rgba(36, 113, 245, 0.10);
  border: 1px solid hsl(var(--primary) / 0.3);
  border-radius: 999px;
  font-size: 12px; color: hsl(var(--primary));
  width: fit-content;
}
```

- [ ] **Step 3: `renderOverview`**

```js
// OVERVIEW
function renderOverview(s) {
  if (!document.getElementById('ovKpiExecs')) return;
  const d = activeDomainData();
  const m = activeMetrics();
  const periodLabel = ({'1d':'ultimas 24 horas','7d':'ultimos 7 dias','30d':'ultimos 30 dias'})[s.period];
  document.getElementById('ovPeriodLabel').textContent = periodLabel;

  animateNumber(document.getElementById('ovKpiExecs'), m.execs, 600, v => v.toLocaleString('pt-BR'));
  document.getElementById('ovKpiTokens').textContent   = m.tokens;
  document.getElementById('ovKpiCost').textContent     = m.cost;
  document.getElementById('ovKpiApproval').textContent = m.approval.toFixed(1) + '%';

  const banner = document.getElementById('ovDomainBanner');
  if (d) {
    banner.style.display = 'inline-flex';
    banner.querySelector('.ov-banner-name').textContent = d.fullName;
  } else {
    banner.style.display = 'none';
  }

  const qcEl = document.getElementById('ovQuickCards');
  qcEl.innerHTML = Object.values(KAIROS_DATA.featured).map(dd => {
    const dm = dd.metrics[s.period];
    return '<article class="ov-quick-card" data-quick="' + dd.id + '">'
      + '<div class="ov-quick-card-eyebrow">' + dd.regulators[0] + '</div>'
      + '<div class="ov-quick-card-title">' + dd.fullName + '</div>'
      + '<div class="ov-quick-card-meta">' + dm.execs.toLocaleString('pt-BR') + ' execs · ' + dm.cost + ' · aprovacao ' + dm.approval.toFixed(1) + '%</div>'
      + '</article>';
  }).join('');
  qcEl.querySelectorAll('.ov-quick-card').forEach(el => {
    el.addEventListener('click', () => {
      setState({ currentDomain: el.dataset.quick });
      appNavigate('playground');
      pgLoad();
    });
  });

  const feedEl = document.getElementById('ovFeedList');
  const recent = (s.ticker.recentTraces.length === 0 ? synthInitialTraces(5) : s.ticker.recentTraces).slice(0, 5);
  feedEl.innerHTML = recent.map(t => {
    const ts = new Date(t.timestamp);
    const hh = String(ts.getHours()).padStart(2, '0');
    const mm = String(ts.getMinutes()).padStart(2, '0');
    return '<div class="ov-feed-row" data-trace-id="' + t.id + '">'
      + '<span class="ov-feed-time">' + hh + ':' + mm + '</span>'
      + '<span class="ov-feed-title"><strong>' + t.domainName + '</strong> · ' + t.scenarioName + '</span>'
      + '<span class="ov-feed-meta"><code>' + (t.durationMs/1000).toFixed(2) + 's</code></span>'
      + '<span class="drill-tag success">' + t.status + '</span>'
      + '</div>';
  }).join('');
  feedEl.querySelectorAll('.ov-feed-row').forEach(el => {
    el.addEventListener('click', () => {
      const trace = recent.find(t => t.id === el.dataset.traceId);
      if (trace) openDrill('trace', trace);
    });
  });
}
subscribe(renderOverview);

document.getElementById('ovBannerClear').addEventListener('click', () => setState({ currentDomain: null }));
```

- [ ] **Step 4: Smoke test**

`A` → Overview:
- 4 KPIs (global, currentDomain null)
- 3 quick cards GovAI featured
- Activity feed com 5 entradas
- Clicar ContrIA quick card → Playground com ContrIA
- Voltar Overview → banner aparece
- Trocar período → KPIs reagem
- Limpar banner → volta pro global

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat(app): overview v2 - hero KPIs + quick cards + activity feed

Co-Authored-By: Claude Opus 4.7 (1M context) <noreply@anthropic.com>"
```

---

## Phase 11 — Ticker de Atividade

### Task 11.1: `setInterval` com auto-pausa

**Files:**
- Modify: `index.html` — adicionar antes do `fitScale()`

- [ ] **Step 1: Adicionar bloco do ticker**

```js
// TICKER - simulacao de atividade ao vivo
const TICK_INTERVAL_MS = 9000;

function shouldTickerRun() {
  if (state.ticker.paused) return false;
  if (state.drillDown) return false;
  if (state.currentScreen === 'playground') return false;
  if (document.body.classList.contains('recording-mode')) return false;
  if (!document.body.classList.contains('app-mode')) return false;
  return true;
}

function tick() {
  if (!shouldTickerRun()) return;

  const inc = 6 + Math.floor(Math.random() * 13);
  KAIROS_DATA.global.metrics[state.period].execs += inc;

  const featured = Object.values(KAIROS_DATA.featured);
  const d = featured[Math.floor(Math.random() * featured.length)];
  const sc = d.scenarios[Math.floor(Math.random() * d.scenarios.length)];
  const trace = {
    id: 't' + Date.now().toString(36) + Math.floor(Math.random() * 99),
    timestamp: Date.now(),
    domain: d.id,
    domainName: d.name,
    agent: d.agents[0].role,
    scenarioId: sc.id,
    scenarioName: sc.name,
    status: 'pass',
    durationMs: 850 + Math.floor(Math.random() * 1100),
    tokens: 1900 + Math.floor(Math.random() * 3300),
    pipeline: d.assertions.filter(a => a.pipeline).slice(0, 4).map(a => ({
      name: a.name,
      description: a.description,
      durationMs: 220 + Math.floor(Math.random() * 260),
      status: 'pass',
    })),
  };
  const list = state.ticker.recentTraces.slice();
  list.unshift(trace);
  if (list.length > 20) list.length = 20;
  setState({ ticker: Object.assign({}, state.ticker, { recentTraces: list, lastTick: Date.now() }) });
}

setInterval(tick, TICK_INTERVAL_MS);
```

- [ ] **Step 2: Smoke test (5 cenários)**

`A` → Observability. Esperar ~9s. Nova trace aparece com fade-in.

- Playground ativo → ticker para
- Voltar Observability → ticker volta
- `R` → recording-mode → ticker para
- `R` → volta
- `T` → toast "Ticker pausado" → ticker para
- `T` → volta
- Drill aberto → ticker para até fechar

Console limpo.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat(app): live activity ticker with auto-pause rules

Co-Authored-By: Claude Opus 4.7 (1M context) <noreply@anthropic.com>"
```

---

## Phase 12 — QA Roteiro + Cleanup

### Task 12.1: QA do roteiro de 6 telas

**Files:**
- Nenhum (verificação manual end-to-end)

- [ ] **Step 1: Servir e abrir**

```bash
python3 -m http.server 8000 &
sleep 1
open http://localhost:8000
```

- [ ] **Step 2: Roteiro completo**

Em viewport 1920×1080 (Chrome, F11 fullscreen):

1. `A` → Overview com KPIs animados, 3 quick cards, activity feed
2. `R` → recording-mode (nav-controls/hint somem; ticker pausa)
3. Menu "Domain Catalog" → 9 cards. Busca "lei" → filtra. Chip "GovAI" → 6 cards. "Todos".
4. Click ContrIA → drill com agentes/assertions/cenarios. "Executar no Playground".
5. Playground: dropdown cenários. Click "Executar". Pipeline 4 steps. Output. Sumário.
6. "Ver na Observabilidade". Banner "Filtrado por...". Trace nova no topo. KPIs filtrados. Troque período. Click trace → drill.
7. FinOps via menu. Custos ContrIA. Budget pulsando. Click Sonnet → drill. Click budget → drill.
8. Compliance via menu. Lei 14.133 badge "aplicável". Items ContrIA. Click → drill.
9. `R` para sair. `Esc` → volta pros slides.

Console limpo o tempo todo. Anotar hiccups.

- [ ] **Step 3: QA exploração livre**

Sem domínio selecionado, clicar:

- Cada item da sidebar (8 telas)
- Cada chip em cada tela
- 5 cards aleatórios do Catálogo
- 3 rows em Agents/Assertions/Observability
- Budget em FinOps
- Cada categoria em Compliance

Zero erros no console. Drill abre/fecha. Esc volta sempre.

- [ ] **Step 4: QA regressão dos Slides**

`Esc` → slides. → → → percorre 9 slides. `R` "Gravando". `F` fullscreen. Tudo intacto.

- [ ] **Step 5: Anotar bugs** (se houver) e tratar antes de seguir.

### Task 12.2: Cleanup do `tmp/`

**Files:**
- Delete: `tmp/dominios-extract/`

- [ ] **Step 1: Remover**

```bash
rm -rf tmp/dominios-extract/
rmdir tmp/ 2>/dev/null || true
```

- [ ] **Step 2: Verificar gitignore**

```bash
git status
```

Expected: working tree clean.

### Task 12.3: Final commit & deploy

**Files:**
- Possivelmente `index.html` (polish residual)

- [ ] **Step 1: Polish (se aplicável)**

Aplicar ajustes pequenos da Task 12.1. Senão, pular.

- [ ] **Step 2: Push**

```bash
git push origin main
```

GitHub Pages workflow é disparado.

- [ ] **Step 3: Validar deploy**

Abrir a URL de produção (verificar em `gh repo view --json url` ou GitHub Settings → Pages). Mini-QA do roteiro.

- [ ] **Step 4: Tag**

```bash
git tag -a v1.0-modo-aplicacao-funcional -m "Modo Aplicacao funcional: estado por dominio, drill-down, ticker, 3 GovAI featured"
git push origin v1.0-modo-aplicacao-funcional
```

---

## Self-Review

**1. Spec coverage:**

| Spec section | Task(s) |
|---|---|
| D1 (Roteiro 6 telas) | Tasks 4.x, 5.x, 6.x, 7.x, 8.x, 10.x |
| D2 (Estado compartilhado) | Task 2.1 + `activeDomainData`/`activeMetrics` em todos renders |
| D3 (Catálogo híbrido) | Task 4.1 |
| D4 (ContrIA + LicitIA + PrivacIA) | Tasks 1.2/1.3/1.4 + 1.5-1.9 |
| D5 (Dados embedados) | Tasks 1.x |
| D6 (Drill + simulação) | Tasks 3.x + 11.x |
| D7 (Agents/Assertions vivos) | Tasks 9.x |
| D8 (Store reativo vanilla) | Task 2.1 |
| Drill panel único | Task 3.x |
| Ticker auto-pausa | Task 11.1 |
| Atalhos T/D/Esc | Task 3.3 |
| Critérios de aceitação 1-7 | Task 12.1 (QA cobre todos) |

**2. Placeholder scan:** ✓ Sem "TBD"/"TODO"/"implement later". Comentários `/* X da curadoria */` em Task 1.9 são intencionais: a curadoria é o trabalho, não placeholder. Cada estrutura alvo está descrita nas Tasks 1.5-1.7.

**3. Type consistency:**
- `state.currentDomain`: `'contria' | 'licitia' | 'privacia' | null` — consistente em todas as tasks.
- `state.period`: `'1d' | '7d' | '30d'` — consistente.
- Trace shape `{id, timestamp, domain, domainName, agent, scenarioId, scenarioName, status, durationMs, tokens, pipeline}` — idêntico em Task 5.2 (`emitTrace`), Task 6.2 (`synthInitialTraces`), Task 11.1 (`tick`).
- `openDrill(type, payload)` types: 8 valores cobertos com renderer concreto (`domain`, `placeholder` em Task 3.2; `trace` em Task 6.2; `model`, `budget` em Task 7.2; `compliance-item` em Task 8.1; `agent` em Task 9.1; `assertion` em Task 9.2).
- Helpers (`activeDomainData`, `activeMetrics`, `formatCurrency`, `sleep`, `animateNumber`, `emitTrace`, `synthInitialTraces`, `showAppToast`) — nomes batem em todos os usos.

**4. Scope check:** Feature coesa em arquivo único; não precisa decomposição.

---

## Execution Handoff

**Plan complete and saved to** `docs/superpowers/plans/2026-05-19-modo-aplicacao-funcional.md`. **Two execution options:**

**1. Subagent-Driven (recommended)** — Despacho subagente fresco por task, revisão entre tasks. Cada task tem todo contexto. Bom para 24 tasks ~10h.

**2. Inline Execution** — Executar nesta sessão com batches de checkpoints.

**Qual abordagem?**
