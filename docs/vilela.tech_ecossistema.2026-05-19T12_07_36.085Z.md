Ecossistema KairOS · 6 camadas

# 6 camadas para construir, operar e governar  software com coding agents em PT-BR

O KairOS é o stack completo da VilelaAI para times que querem usar Claude Code, Codex CLI e OpenCode com qualidade e governança. Vai do open-source MIT até o backend SaaS regulado — você adota o que precisa, quando precisa.

8

produtos no ecossistema

34

domínios regulados

2

licenças (MIT + PRO)

[Ver as 6 camadas](https://vilela.tech/ecossistema#camadas) [Como começar](https://vilela.tech/ecossistema#comecar)

## O que é o KairOS?

O KairOS é o stack de produtos da VilelaAI para times que constroem software com coding agents (Claude Code, Codex CLI, OpenCode). Não é um único produto — é um ecossistema com **6 camadas independentes mas integradas**.

Você pode adotar **só uma camada** (ex: o kairos-forge MIT no repo do seu time) ou **o stack todo** (Foundation + Domains + Runtime + Orchestration + Operations + Products).

A premissa central, inspirada no [Harness Engineering da OpenAI](https://openai.com/index/harness-engineering/) (fevereiro 2026): **a qualidade do output do agente depende mais do ambiente do que do prompt**. Cada camada do KairOS prepara um pedaço desse ambiente.

### Camadas independentes

Cada produto resolve um problema específico. Não força você a adotar tudo.

### MIT + PRO claros

A porta de entrada é open-source MIT. Componentes regulados/enterprise são PRO.

### PT-BR oficial

Personas, agentes, skills e documentação em português brasileiro.

## As 6 camadas do ecossistema

Cada camada acima depende do que tem abaixo. Foundation é o que você instala primeiro; Products é o que você entrega no final.

O que o cliente recebeO que instalamos primeiro06ProductsOs produtos finais que o seu time usa no dia a diaEm pipeline05OperationsAcompanha, audita e governa o que a IA fazEm desenvolvimento04OrchestrationCoordena vários agentes trabalhando juntos, 24/7Especificação aberta03RuntimeO motor que faz a IA rodar de verdadeEm desenvolvimento02DomainsConhecimento do seu setor (leis, normas, processos)Disponível01FoundationA base: prepara o ambiente onde a IA vai trabalharDisponível

- 01


Foundation



[kairos-forgeMIT](https://vilela.tech/kairos-forge) [kairos-aiPRO](https://vilela.tech/kairos)

- 02


Domains



kairos-domains — 34 domíniosPRO

- 03


Runtime



kairos-runtimeMIT

- 04


Orchestration



[kairos-symphonyMIT](https://github.com/VilelaAI/kairos-symphony)

- 05


Operations



kairos-platform (backend)PROkairos-studio (SDK cliente)PRO

- 06


Products



Inclua, JurixIA, Labore, NumerIA, PrivacIA, …


Empilhamento de baixo para cima: Foundation é o que você instala primeiro; Products é o que você entrega no final. Cada camada depende do que está abaixo.

Camada 1 · Foundation

## Foundation — Preparar repo e dar capacidades ao agente

A base de tudo. Define como seu repositório fica pronto pra ser operado por coding agents.

### kairos-forge

MIT

Harness + Persona Library para qualquer projeto.

- 5 pilares de harness (Repository-as-context, Instruction Set, Invariants, Observability, Iteração autônoma)
- 45 personas curadas em PT-BR (24 core + 21 apoio)
- 8 skills cobrindo o ciclo completo
- Multi-CLI: Claude Code, Codex CLI, OpenCode

[Conhecer o kairos-forge](https://vilela.tech/kairos-forge)

### kairos-ai

PRO

Forge + scaffolding regulado + squads negociais brasileiros.

- Tudo do kairos-forge MIT (herda os 5 pilares + 45 personas)
- Squads negociais regulados (DPO, Mapeamento, Consentimento)
- Guardrails com referência legal explícita
- Ralph Loop com 3 retries auto-corretivos
- 31 skills (vs 8 do Forge)
- Advisor regulatório (Opus para decisões críticas)

[Conhecer o kairos-ai PRO](https://vilela.tech/kairos)

Camada 2 · Domains

## Domains — Conhecimento estruturado por domínio

34 domínios proprietários com agentes, scenarios e assertions binárias vinculadas à legislação brasileira.

### kairos-domains

PRO

#### KairOS-PRO — 28 domínios setoriais

Divididos em **Core 10** \+ **Marketplace 18**.

**Core 10:** LGPD, Segurança TI, SST (NRs), Jurídico, Saúde (ANVISA), Educação (BNCC/LDB), Contábil/Fiscal (NBC TG, SPED), Trabalhista (CLT), Financeiro (BACEN), Inclusão (LBI).

**Marketplace 18:** customização por nicho (energia, telecom, agro, etc.)

192

agentes negociais

223

scenarios

478

assertions

#### KairOS-GovAI — 6 módulos governo

6 módulos específicos para o setor público brasileiro.

- **TransparêncIA** — Lei 12.527/2011 (LAI) e Lei 13.460/2017
- **LicitIA** — Lei 14.133/2021 (nova lei de licitações)
- **ContrIA** — gestão e fiscalização contratual
- **ServidorIA** — regime jurídico do servidor (Lei 8.112/90)
- **OuvidorIA** — manifestações cidadãs (Lei 13.460/2017)
- **DadosAbertosIA** — Decreto 8.777/2016 e portarias INDA

35

agentes

52

scenarios

119

assertions

[Conhecer o kairos-domains](https://vilela.tech/kairos-domains) Falar com vendas

Camada 3 · Runtime

## Runtime — Como o agente executa

Dois caminhos paralelos: agente rodando dentro de CLI (dev local) ou agente rodando dentro de aplicação (produto em produção).

### CLIs (Claude Code, Codex, OpenCode)

Terceiros

O kairos-forge e kairos-ai são plugins desses CLIs. Você instala o plugin e usa via comandos `/kairos-forge:*` ou `/kairos:*`.

- • Claude Code (Anthropic)
- • Codex CLI (OpenAI)
- • OpenCode (community)

Compatível hoje [Como instalar o plugin](https://vilela.tech/kairos-forge)

### kairos-runtime

MIT

Biblioteca NPM para apps Next.js / Edge Function chamarem agentes diretamente (sem precisar de CLI).

- • Vercel AI SDK v5 + OpenRouter
- • Three-tier model routing (fast/balanced/powerful)
- • Ralph Loop assertion retry integrado
- • Export `/edge` separado para Supabase Edge Functions

Em desenvolvimento [Conhecer o kairos-runtime](https://vilela.tech/kairos-runtime)

Camada 4 · Orchestration

## Orchestration — Operação contínua (agentes always-on)

Quando você quer agentes trabalhando 24/7 sobre o seu issue tracker, ao invés de invocar via CLI manualmente.

### kairos-symphony

MITSpec aberta

Orquestrador always-on que transforma issue tracker em state machine. Inspirado no Symphony da OpenAI (abril/2026), adaptado para multi-tracker e multi-CLI.

#### Como funciona

Daemon polla seu tracker (GitHub Issues, Jira, Linear, GitLab). Identifica issues no estado "ready". Spawna agente dedicado em git worktree isolado. Monitora execução. Atualiza estado da issue conforme progresso. Você revisa PRs prontos.

#### Diferenças vs Symphony OpenAI

- • Multi-tracker (não só Linear)
- • Multi-CLI (não só Codex)
- • Reference em Node/TS (não Elixir)
- • Pré-requisito: harness-readiness validado
- • PT-BR oficial nos logs

Spec v0.3.2 publicada. Reference implementation em planejamento.

[Ver SPEC no GitHub](https://github.com/VilelaAI/kairos-symphony) Falar com especialista

Camada 5 · Operations

## Operations — Observability + governança

A camada de runtime de produção. Backend SaaS de governança recebe telemetria dos produtos via SDK cliente.

### kairos-platform

PRO

Backend SaaS multi-tenant. **Servidor.**

- **Observabilidade** — telemetria centralizada de agentes
- **FinOps** — custo de tokens por agente/produto/cliente
- **AIOps** — drift, qualidade, regressões
- **AI Studio** — interface humana sobre agentes em operação

Em desenvolvimento (protótipo) [Conhecer o kairos-platform](https://vilela.tech/kairos-platform)

### kairos-studio

PRO

SDK cliente que produtos VilelaAI consomem pra integrar com a platform. **Cliente.**

- `@kairos.ai/studio-ui` — React components (design-time)
- `@kairos.ai/studio-sdk` — NestJS interceptor (runtime telemetry)
- `studio.kairos.ai` — Next.js app cross-product

Pacotes NPM publicados [Conhecer o kairos-studio](https://vilela.tech/kairos-studio)

**Relação cliente ↔ servidor:** o kairos-studio é importado pelos produtos VilelaAI (Inclua, JurixIA, etc.) como SDK. Ele emite telemetria via @kairos.ai/studio-sdk que vai pro kairos-platform. O studio.kairos.ai (Next.js app) consome essa telemetria pra dar dashboards de ops cross-produto.

Camada 6 · Products

## Products — Saídas finais aplicando todo o stack

Produtos verticais VilelaAI construídos sobre as 5 camadas anteriores. Cada um endereça um domínio regulado específico.

[Em pilot (Prefeitura de Natal) **Inclua** \\
\\
AI para educação inclusiva em redes municipais\\
\\
Stack\\
\\
`kairos-ai + kairos-domains/educacao + kairos-runtime + kairos-studio`\\
\\
Planos de ensino adaptados, identificação de NEE, laudos pedagógicos com base na LDB\\
\\
Conhecer Inclua](https://vilela.tech/inclua)

[Em desenvolvimento **JurixIA** \\
\\
AI para análise e elaboração de contratos jurídicos\\
\\
Stack\\
\\
`kairos-ai + kairos-domains/juridico + kairos-runtime`\\
\\
Revisão de contratos, identificação de cláusulas de risco, geração de minutas\\
\\
Conhecer JurixIA](https://vilela.tech/jurixia)

[Roadmap **Labore** \\
\\
AI para departamento pessoal e direito trabalhista (CLT)\\
\\
Stack\\
\\
`kairos-ai + kairos-domains/trabalhista + kairos-runtime`\\
\\
Cálculos trabalhistas, conformidade CLT, automação de processos DP\\
\\
Conhecer Labore](https://vilela.tech/labore)

[Roadmap **NumerIA** \\
\\
AI para contabilidade e fiscal (NBC TG, eSocial, SPED)\\
\\
Stack\\
\\
`kairos-ai + kairos-domains/contabil + kairos-runtime`\\
\\
Apuração assistida, conformidade fiscal, conciliação contábil automatizada\\
\\
Conhecer NumerIA](https://vilela.tech/numeria)

[Roadmap **PrivacIA** \\
\\
AI para programa de privacidade e LGPD operacional\\
\\
Stack\\
\\
`kairos-ai + kairos-domains/lgpd + kairos-runtime + kairos-platform`\\
\\
RIPD, mapeamento de fluxo, atendimento de titulares, registros LGPD Art. 37\\
\\
Conhecer PrivacIA](https://vilela.tech/privacia)

[Roadmap **MedixIA** \\
\\
AI para regulatório de saúde (ANVISA, RDC)\\
\\
Stack\\
\\
`kairos-ai + kairos-domains/saude`\\
\\
Conformidade RDC, dossiê regulatório, vigilância pós-mercado\\
\\
Conhecer MedixIA](https://vilela.tech/medixia)

Produtos adicionais em pipeline: LicitIA (licitações públicas), RiskIA (NR-01 e gestão de riscos), EducaSmart (educação básica BNCC), AutonomIA (automação de processos).

[Ver todas as soluções](https://vilela.tech/solucoes) Falar com vendas

## Quem combina com o quê

Você não precisa adotar tudo. Veja stacks típicas conforme seu caso de uso.

| Caso de uso | Stack típica |
| --- | --- |
| Dev solo, repo MIT genérico | `kairos-forge plugin no Claude Code/Codex` |
| Time pequeno, produto regulado | `kairos-ai + kairos-domains (1-2 domínios)` |
| Aplicação Next.js que chama agente em runtime | `kairos-runtime no backend` |
| Time eng com tracker (GitHub) e quer agentes 24/7 | `kairos-forge no repo + kairos-symphony no devbox` |
| Produto VilelaAI integrado à plataforma de governança | `kairos-studio (SDK no produto) + kairos-platform (backend)` |
| Empresa com múltiplos produtos KairOS | `Stack completa: ai + domains + runtime + symphony + studio + platform` |

## O que é MIT, o que é PRO

Você começa pelo MIT e adota PRO quando precisar.

| Componente | Licença | O que faz |
| --- | --- | --- |
| kairos-forge | MIT | Harness + Persona Library para qualquer projeto. Porta de entrada. |
| kairos-runtime | MIT | Biblioteca NPM para apps chamarem agentes. |
| kairos-symphony | MIT | Orquestrador always-on. Spec aberta, reference em Node/TS. |
| kairos-ai | PRO | Forge + scaffolding regulado + squads negociais brasileiros. |
| kairos-domains | PRO | 34 domínios proprietários: 28 setoriais + 6 governo. |
| kairos-platform | PRO | Backend SaaS multi-tenant de governança. |
| kairos-studio | PRO | SDK cliente que produtos consomem para integrar com platform. |
| Produtos VilelaAI | Diversos | Construídos sobre o stack. Cada produto tem licenciamento próprio. |

## Roadmap geral

O que está pronto, o que está em desenvolvimento, o que vem depois.

### Q4 2025 — Foundations

- kairos-ai v0.2 (MIT na época)
- KairOS-PRO domínios iniciais

### Q1 2026 — Reposicionamento

- Separação kairos-forge MIT vs kairos-ai PRO
- 5 pilares de harness explicitados (ADR-0005, ADR-0006)
- kairos-symphony SPEC v0.3.2
- kairos-studio (pacotes NPM publicados)

### Q2 2026 — Ativação comercial (você está aqui)

- kairos-runtime v1.0
- kairos-platform MVP
- Inclua pilot Prefeitura de Natal
- JurixIA beta

### Q3 2026 — Expansão

- kairos-symphony reference implementation
- kairos-domains marketplace público
- KairOS-GovAI primeiros módulos disponíveis
- Labore beta

### Q4 2026 e além

- kairos-platform multi-tenant production
- Stack completa em 3+ clientes enterprise
- Marketplace de domínios da comunidade

## Como começar — escolha seu caminho

O KairOS é modular. Você não precisa entender tudo pra começar.

### Quero testar agora

Dev curioso · gratuito · 10 min

1. 1\. Instale Claude Code ou Codex CLI
2. 2\. Adicione o marketplace `VilelaAI/kairos-forge`
3. 3\. Rode `/kairos-forge:onboardar` em um projeto seu

[Ir para kairos-forge MIT](https://vilela.tech/kairos-forge)

### Tenho compliance no projeto

Time regulado · conversa em 1 dia

1. 1\. Avalie comparativo Forge MIT vs ai PRO
2. 2\. Fale com o time da VilelaAI
3. 3\. Projeto piloto em 2 semanas

Falar com o time

### Quero o stack todo

Stack completa · parceria estratégica

1. 1\. Apresente caso de uso e volume
2. 2\. Roadmap conjunto com VilelaAI
3. 3\. POC com kairos-ai + domains + studio

Solicitar apresentação

## O ecossistema KairOS está aberto

Comece pelo kairos-forge MIT no GitHub. Evolua conforme sua necessidade. A VilelaAI está construindo o stack completo em PT-BR pra times que querem qualidade e governança em projetos com coding agents.

MIT

### kairos-forge

Plugin open-source de harness e personas.

[Ver no GitHub](https://github.com/VilelaAI/kairos-forge)

PRO

### kairos-ai

Forge regulado + squads negociais.

[Falar com o time](https://vilela.tech/kairos)

MIT

### kairos-symphony

Orquestrador always-on (spec aberta).

[Ver SPEC](https://github.com/VilelaAI/kairos-symphony)

Empresa

### VilelaAI

Construímos o stack todo em PT-BR.

[Conhecer a empresa](https://vilela.tech/)