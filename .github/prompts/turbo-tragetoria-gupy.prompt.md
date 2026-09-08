---
description: Gera um resumo profissional otimizado para candidaturas na Gupy, cruzando a descrição de uma vaga com o currículo de Gabriel Vieira e destacando a trajetória mais compatível com aquela vaga.
argument-hint: Cole a descrição completa da vaga (requisitos, responsabilidades, palavras-chave)
---

# Turbo Trajetória Gupy

Gera um **resumo profissional** pronto para colar no campo "Resumo Profissional" da Gupy, otimizado para compatibilidade currículo x vaga (palavras-chave, experiências, competências).

> **Escopo:** este prompt cobre apenas o critério de **compatibilidade currículo x vaga**. Não analisa requisitos eliminatórios nem perguntas obrigatórias da candidatura, e não sinaliza gaps/riscos do perfil — foco 100% em destacar pontos fortes e aderência.

---

## Entrada

- **Vaga:** descrição da vaga fornecida pelo usuário como argumento/contexto (requisitos, responsabilidades, stack, palavras-chave).
  - Se o usuário não colar a descrição da vaga, pare e peça que ele cole antes de continuar.
- **Currículo:** sempre ler o arquivo fixo [docs/linkedIn/curriculo-gabriel-vieira.md](../../../../../source/repos/v-tools/docs/linkedIn/curriculo-gabriel-vieira.md) por completo antes de gerar qualquer resumo.

## Passo 1 — Extrair sinais da vaga

Leia a descrição da vaga e identifique:
- Palavras-chave técnicas (linguagens, frameworks, ferramentas, metodologias).
- Nível de senioridade esperado.
- Principais responsabilidades/entregas do cargo.
- Diferenciais mencionados como desejáveis (não obrigatórios).

## Passo 2 — Cruzar com o currículo

Leia [docs/linkedIn/curriculo-gabriel-vieira.md](../../../../../source/repos/v-tools/docs/linkedIn/curriculo-gabriel-vieira.md) e identifique quais experiências, stacks e conquistas do candidato têm maior aderência aos sinais extraídos no Passo 1. Priorize:
- Experiências e tecnologias que aparecem tanto na vaga quanto no currículo.
- Resultados/conquistas quantificáveis que reforcem esses pontos.
- Trajetória (evolução de cargo, autonomia, senso de dono) quando relevante para o nível da vaga.

Ignore, para fins deste resumo, qualquer requisito da vaga que o candidato não atenda — não liste gaps nem faça alertas de risco.

## Passo 3 — Redigir o resumo profissional

Escreva **uma única versão final** do resumo profissional, em português, primeira pessoa, tom direto e profissional (mesmo estilo do resumo já presente no currículo), com as seguintes regras:

- **Tamanho-alvo: até 1200 caracteres** (contando espaços).
- Incorporar naturalmente as palavras-chave da vaga identificadas no Passo 1, sem parecer lista forçada de termos.
- Destacar as experiências e competências do currículo com maior aderência à vaga (Passo 2).
- Não inventar experiências, tecnologias ou números que não estejam no currículo.
- Não incluir análise de requisitos eliminatórios, perguntas obrigatórias ou riscos — apenas o texto do resumo.

## Passo 4 — Selecionar as 3 habilidades mais valiosas

A Gupy costuma pedir para escolher **até 3 habilidades cadastradas no currículo** para destacar ao recrutador (ex.: ".NET", "Azure", "Microsserviços"). Com base nos sinais da vaga (Passo 1) e nas habilidades técnicas do currículo (seção "Habilidades Técnicas"), escolha as **3 habilidades** com maior aderência, priorizando nesta ordem:

1. Hard skills marcadas como obrigatórias/essenciais na vaga.
2. Hard skills que aparecem tanto na vaga quanto no currículo com maior frequência/destaque.
3. Soft skills ou diferenciais, apenas se não houver 3 hard skills claras o suficiente.

Não sugira habilidades que não estejam no currículo do candidato.

## Saída

Entregue apenas:
1. O texto final do resumo profissional (pronto para copiar e colar na Gupy).
2. A contagem de caracteres do texto gerado.
3. As 3 habilidades escolhidas para destacar na Gupy, com uma justificativa breve (1 linha) para cada uma.
