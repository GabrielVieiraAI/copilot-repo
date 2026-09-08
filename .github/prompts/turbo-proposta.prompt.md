---
description: Gera um proposta.md completo para uma nova feature — clarifica o contexto via perguntas guiadas e depois gera e salva o arquivo automaticamente no OpenSpec.
---

# Turbo Proposta

Combina dois comportamentos em sequência:
1. **Fase de Clarificação** — coleta as informações essenciais via perguntas guiadas (`vscode_askQuestions`)
2. **Fase de Geração** — produz e salva o `proposta.md` completo em `openspec/changes/<data-slug>/proposta.md`

> **⚠️ Este prompt gera apenas artefatos OpenSpec.** Nunca escreva código de aplicação.

---

## Fase 1 — Clarificação (SEMPRE primeiro)

### Passo 1.1 — Analisar o Input

Leia com atenção o que o usuário trouxe. Não faça perguntas ainda.

Classifique cada informação abaixo:
- **Resolvido** — informação explícita, sem dupla interpretação
- **Ambíguo** — informação existe, mas pode ser mal interpretada
- **Ausente** — informação não foi fornecida

| Informação necessária | Obrigatoriedade |
|---|---|
| Nome da feature | Obrigatório |
| Descrição em 2–3 frases (o que é e o que faz) | Obrigatório |
| Motivação e problema que resolve | Obrigatório |
| Entidades principais e seus relacionamentos | Obrigatório |
| O que está fora do escopo desta entrega | Obrigatório |
| Flags contextuais (ver lista abaixo) | Obrigatório |
| Decisões já tomadas | Opcional |

### Passo 1.2 — Perguntar sobre Lacunas

> **⚠️ OBRIGATÓRIO:** Toda e qualquer pergunta ao usuário **DEVE** ser feita exclusivamente pela ferramenta `vscode_askQuestions`. É **PROIBIDO** escrever perguntas como texto livre. Sem exceções.
> Toda pergunta feita pelo `vscode_askQuestions` **DEVE** ter uma opção para o usuário escrever livremente, caso nenhuma das opções predefinidas se encaixe.

- Vá direto às perguntas — não repita o contexto antes de perguntar
- **Máximo de 5 a 7 perguntas por rodada**
- Após cada rodada, repita a análise do Passo 1.1 com as novas informações
- Continue o ciclo até que todas as informações obrigatórias estejam resolvidas
- Quando todas estiverem resolvidas, avance para a Fase 2

**Perguntas obrigatórias (fazer quando ausentes):**

1. **Nome da feature** — será usado para derivar o slug da pasta
2. **Descrição** — o que é essa feature e o que o usuário consegue fazer com ela?
3. **Motivação** — qual problema ou necessidade ela resolve?
4. **Entidades** — quais são as principais entidades de dados? Liste brevemente cada uma e seus relacionamentos.
5. **Fora do escopo** — o que esta entrega deliberadamente não cobre?
6. **Flags contextuais** (pergunta multi-select com `multiSelect: true`) — quais características se aplicam a esta feature?

   Opções de flags:
   - `Tem interface de usuário (telas, modais, formulários)`
   - `Dados variam ao longo do tempo (histórico, snapshots, versões, ciclos)`
   - `Há cálculos, totais ou valores derivados`
   - `O comportamento no primeiro acesso é distinto do uso normal (estado vazio/onboarding)`
   - `A identidade visual ou stack de UI está sendo definida nesta entrega`
   - `Há regras de autorização ou permissões por perfil de usuário`

---

## Fase 2 — Geração

Com todas as informações coletadas:

### Passo 2.1 — Preparação

1. Leia `openspec/config.yaml` para obter o contexto e convenções do projeto
2. Derive o slug: nome da feature em lowercase, espaços → hífens, sem acentos ou caracteres especiais
3. Monte o caminho de destino: `openspec/changes/<YYYY-MM-DD>-<slug>/proposta.md` usando a data atual

### Passo 2.2 — Estrutura do proposta.md

Gere o arquivo seguindo esta estrutura. **Os 8 pilares universais são sempre obrigatórios.** As seções contextuais aparecem apenas quando o gatilho correspondente foi confirmado nas flags.

---

#### Cabeçalho (sempre)

```markdown
# Proposta de Implementação — <Nome da Feature>

> **Status:** Em elaboração
> **Data:** <data atual DD/MM/AAAA>
> **Origem:** <como surgiu: sessão de exploração, demanda do usuário, etc.>
> **Módulo:** <nome do módulo ou sistema ao qual pertence>
```

---

#### Pilar 1 — Contexto e Motivação (sempre)

Responde: por que essa feature existe? Qual dor resolve? Para quem é útil? O que o usuário consegue fazer que antes não conseguia?

---

#### Pilar 2 — O que é `<Nome da Feature>` (sempre)

Explica o conceito central em 2–3 parágrafos. Inclui a "metáfora mental" que define a feature (ex: *"uma foto do patrimônio em um determinado momento"*) e deixa explícito o que ela **não é**, para evitar confusão de escopo.

---

#### [CONTEXTUAL] Modelo Temporal — gatilho: flag `dados variam ao longo do tempo`

Descreve como os dados evoluem no tempo: períodos, ciclos de fechamento/reabertura, imutabilidade histórica, estado "aberto" vs. "fechado", o que pode ser editado retroativamente e o que não pode.

Use tabela e/ou diagrama ASCII de máquina de estados.

---

#### Pilar 3 — Regras de Negócio (sempre)

Lista as regras invariáveis, restrições e comportamentos. Separa o que acontece **automaticamente** do que é **acionado pelo usuário**. Inclui máquinas de estado quando há ciclos de vida relevantes.

---

#### [CONTEXTUAL] Cálculos — gatilho: flag `há cálculos, totais ou valores derivados`

Tabela com todos os valores calculados:

| Valor | Cálculo | Observação |
|---|---|---|

---

#### Pilar 4 — Entidades de Dados (sempre)

Tabela: Entidade | Descrição | O que não armazena (quando relevante).

Inclui relacionamentos entre entidades. Torna explícito o que cada entidade **não** é responsável por armazenar, quando isso evita mal-entendidos.

---

#### Pilar 5 — Telas e Fluxo de Usuário (sempre)

Lista as telas ou fluxos previstos com nome, rota (se aplicável) e responsabilidade principal de cada um.

Se não há UI, descreve o fluxo de API, processo ou integração equivalente.

---

#### [CONTEXTUAL] Layout das Telas — gatilho: flag `UI com estrutura nova/complexa`

Diagramas ASCII do layout das telas principais. Use apenas quando ajuda a clarificar o pensamento — não gere ASCII para layouts triviais.

---

#### [CONTEXTUAL] Primeiro Acesso / Estado Vazio — gatilho: flag `primeiro acesso distinto`

Descreve o comportamento quando não há dados: empty state, mensagem de boas-vindas, onboarding, configuração inicial. O que o usuário vê e o que pode fazer.

---

#### Pilar 6 — Operações por Entidade (sempre)

Para cada entidade, uma tabela:

| Operação | Como é acionada | Comportamento |
|---|---|---|

Inclui os campos dos formulários quando há UI (nome, tipo, obrigatoriedade).

---

#### [CONTEXTUAL] Design e Interface — gatilho: flag `identidade visual sendo definida`

Stack de UI, identidade visual, paleta de cores, tipografia, padrões de interação (formulários, feedback, confirmação de ações destrutivas).

---

#### Pilar 7 — Fora do Escopo (sempre)

Tabela explícita do que esta entrega **deliberadamente não cobre**:

| Item | Motivo do adiamento |
|---|---|

---

#### [CONTEXTUAL] Permissões e Autorização — gatilho: flag `regras de autorização por perfil`

Descreve perfis de usuário, quais ações cada perfil pode executar e o que acontece ao tentar acessar algo sem permissão.

---

#### Pilar 8 — Decisões Tomadas (sempre)

Tabela de todas as decisões de produto e design já resolvidas:

| Decisão | Escolha | Justificativa |
|---|---|---|

---

### Passo 2.3 — Geração e Salvamento

1. Gere o conteúdo completo seguindo a estrutura acima
2. Infira o máximo possível das informações coletadas — não deixe seções com conteúdo trivialmente vazio
3. Para trechos sem informação suficiente, use `[TODO: descrição do que falta]` para o usuário completar depois
4. Omita seções contextuais que não têm gatilho ativo — não deixe seções em branco
5. Salve o arquivo em `openspec/changes/<YYYY-MM-DD>-<slug>/proposta.md`
6. Confirme com o caminho completo do arquivo criado

---

## Guardrails

- **Nunca implemente código de aplicação** — apenas artefatos OpenSpec
- **Nunca omita pilares universais** — os 8 pilares aparecem em toda proposta
- **Nunca inclua seções contextuais sem gatilho** — gere apenas o que se aplica
- **Nunca use placeholders genéricos** — infira o máximo, marque com `[TODO]` o que precisar de completamento humano
- **Nunca salve sem confirmar o caminho** — mostre ao usuário onde o arquivo foi criado
- **Sempre derive o slug automaticamente** — não pergunte o nome da pasta, apenas o nome da feature
- **Sempre leia o config.yaml** — o contexto do projeto informa o tom e as convenções da proposta
