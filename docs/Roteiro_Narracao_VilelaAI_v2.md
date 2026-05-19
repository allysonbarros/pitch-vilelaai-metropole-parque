# Roteiro de Narração · VilelaAI

**Submissão ao Edital 01/2026 Metrópole Parque · IMD/UFRN**
**Duração-alvo: 6:50 – 7:10**

Narração: Allyson Vilela (CTO e Principal Investigator)

---

## Cena 1 · Abertura

**Tempo: 0:00 – 0:30 · ~25s de fala**

Em 2023, um advogado trabalhista usou IA para fundamentar uma petição.

A IA citou três jurisprudências — duas delas não existiam.

Em educação especial, professores recebem laudos pedagógicos gerados por IA sem nenhuma trilha de auditoria.

Em órgãos públicos, decisões algorítmicas afetam cidadãos sem que ninguém saiba como foram tomadas.

Eu sou Allyson Vilela, CTO da VilelaAI.

Construímos a camada de governança e orquestração que organizações em jurídico, saúde, educação e governo precisam para usar IA sem violar a lei, perder auditoria ou expor seus clientes.

---

## Cena 2 · Três falhas estruturais

**Tempo: 0:30 – 1:05 · ~30s de fala**

O problema é estrutural.

Primeiro: a IA opera como caixa-preta.

Decisões algorítmicas sem explicação, sem trilha de auditoria, sem responsável institucional. Órgãos de controle simplesmente não conseguem auditar.

Segundo: viés sem detecção.

Sistemas de IA reproduzem discriminação sem que ninguém perceba. Não há testes de equidade, monitoramento sistemático ou documentação.

Terceiro: compliance manual e reativo.

Conformidade legal verificada depois do fato, por humanos sobrecarregados. Quando o problema aparece, o dano já foi feito.

A VilelaAI resolve isso embarcando governança na infraestrutura. Verificada antes — não depois.

---

## Cena 3 · Arquitetura · seis camadas

**Tempo: 1:05 – 2:00 · ~50s de fala**

Construímos o KairOS, organizado em seis camadas.

Na base, Foundation — o kairos-forge MIT, open-source, porta de entrada gratuita.

Acima, Domains: catálogo de 34 domínios regulados brasileiros, 478 assertions com base legal explícita.

Runtime e Operations já têm pacotes publicados no NPM — três pacotes validados em produção.

Orchestration entra em fase de testes.

E no topo, KairOS-GovAI: governança algorítmica para o setor público brasileiro, alinhada aos Princípios de IA da OCDE — é onde está nosso vetor de escala.

Foundation e Domains: disponíveis hoje. Runtime e Operations: MVP no ar. Orchestration: em testes. GovAI: a próxima fronteira da empresa, mapeado em sete domínios da gestão pública brasileira.

---

## Cena 4 · Transição para a demonstração

**Tempo: 2:00 – 2:25 · ~22s de fala**

O que vou demonstrar agora é o KairOS Studio em operação.

Três capacidades integradas: o catálogo de domínios regulados — 34 domínios, 192 agentes, 478 assertions com base legal; o pipeline de anti-alucinação em quatro camadas que valida toda saída antes do usuário; e a observabilidade multi-tenant com FinOps e compliance em tempo real.

Vamos ao vivo.

---

## Cena 5 · Demonstração ao vivo

**Tempo: 2:25 – 5:00 · ~2:35 de demonstração**

> Esta cena é narrada em fragmentos, conforme a navegação no Modo Aplicação do KairOS Studio. Cada bloco abaixo corresponde a uma tela.

### Overview (~15s)

Esta é a tela inicial do Studio. Atividade em tempo real através de todos os tenants em produção — pipelines aprovados, novos agentes publicados, relatórios gerados.

### Catalog (~30s)

Catálogo de Domínios Regulados. 34 domínios brasileiros mapeados — jurídico, trabalhista, tributário, saúde, educação inclusiva, e mais 29.

Cada domínio carrega agentes especializados, cenários canônicos e assertions vinculadas à legislação.

Isso é o que organiza nossa pesquisa aplicada.

### Agents e Assertions (~40s)

Cada agente é versionado, com prompts evolutivos e métricas de aprovação.

E cada assertion tem base legal explícita — Art. 28 da Lei Brasileira de Inclusão, Art. 477 da CLT, Art. 18 da LGPD.

Não é metáfora — é referência rastreável que vincula a saída do agente a uma norma específica.

### Playground (~50s · cena central — não acelerar)

Aqui está uma execução real.

Cenário de educação inclusiva, aluno de 4º ano com TEA e dislexia.

Pressiono Executar.

A saída do agente passa por quatro camadas sucessivas de validação:

Estrutural — a forma está correta.

Factual — referências à BNCC e à LBI são reais.

Regulatória — atende ao Art. 28 da Lei Brasileira de Inclusão.

Pragmática — linguagem apropriada ao destinatário.

Tudo em três segundos. Custo: sete centavos de real.

Esta é a contribuição técnica central do KairOS — anti-alucinação em quatro camadas como gate obrigatório.

### Observability (~25s)

Toda execução é observada.

Quatro tenants ativos, filtros temporais.

Custo por tenant em tempo real, taxa de aprovação por domínio, drift detectado.

Isolamento estrutural por inquilino via AsyncLocalStorage.

### FinOps (~15s)

Custo broken down por modelo.

Sonnet 4.6 leva 62%. Haiku 4.5 leva 21%. GPT-4o leva 11%. Gemini leva 6%.

Projeção mensal automaticamente calculada.

### Compliance (~15s)

E aqui geramos automaticamente relatórios para ANPD, MPT, eSocial.

Cada documento com trilha de auditoria SHA-256 imutável.

Quando o auditor pedir, é um clique.

---

## Cena 6 · Números que sustentam a tese

**Tempo: 5:00 – 5:40 · ~35s de fala**

Os números que sustentam a tese.

Do lado do ecossistema: oito produtos no KairOS, trinta e quatro domínios regulados brasileiros, quatrocentas e setenta e oito assertions com base legal, três pacotes NPM já publicados e validados em produção.

Do lado da empresa: cento e trinta e um mil reais faturados em 2026, em apenas quatro meses — superando todo o ano de 2025, com oitenta e nove por cento de mercado externo.

RBT12 — receita dos últimos doze meses — em cento e setenta e três mil reais, setenta e sete por cento externo.

Doze patentes INPI do pesquisador principal.

E três apoios institucionais — AWS Activate, Microsoft for Startups, NVIDIA Connect.

---

## Cena 7 · Três pilares hierárquicos

**Tempo: 5:40 – 6:15 · ~35s de fala**

Operamos em três pilares hierárquicos.

Hoje: Engenharia de IA Aplicada — serviços profissionais sobre o KairOS.

É a fonte atual de receita, financia a pesquisa e valida a infraestrutura em casos reais.

Doze meses: KairOS como infraestrutura licenciada.

Modelo open-core inspirado em Red Hat — Foundation MIT pública, camadas avançadas em PRO.

Três NPMs já no ar permitem que essa oferta deixe de ser horizonte e vire produto.

Vinte e quatro a trinta e seis meses: KairOS-GovAI.

Plataforma multi-agente para governança algorítmica no setor público brasileiro.

TAM de cinco mil quinhentos e setenta municípios mais vinte e sete estados.

É o vetor de escala e o motivo pelo qual a captação seed está sendo estruturada.

---

## Cena 8 · A equipe que executa

**Tempo: 6:15 – 6:35 · ~20s de fala**

Quem está por trás.

Allyson Vilela — eu — Co-Fundador e CTO. Mestre em Engenharia de Software pela UFRN, servidor do IFRN, doze patentes INPI.

Anderson Vilela, co-fundador e CEO, lidera a engenharia de produto.

Filipe Vilela, Legal Product Manager, formado em Direito pela UFRN — traduz requisitos legais em especificações de agentes.

Três fundadores: arquitetura, engenharia e direito.

---

## Cena 9 · Pedidos e encerramento

**Tempo: 6:35 – 7:00 · ~25s de fala**

O que pedimos ao Metrópole Parque.

Primeiro: mentoria em open-core e captação seed para acelerar o KairOS como plataforma licenciada.

Segundo: credencial institucional IMD/UFRN para abrir portas em clientes corporativos e órgãos públicos.

Terceiro: acesso a uma rede de prefeituras-piloto no Rio Grande do Norte para validar o GovAI em três a cinco municípios reais.

Quarto: Data Center com condições diferenciadas para hospedar a operação multi-tenant da plataforma e do GovAI.

Viemos do IFRN, da UFRN, da UFG.

Voltamos agora, pelo Metrópole Parque, para construir aqui no Rio Grande do Norte a infraestrutura de IA que os domínios regulados brasileiros precisam.

Somos VilelaAI.

---

*Vilela Technology Ltda · CNPJ 58.821.922/0001-40 · Recife/PE · vilela.tech*
