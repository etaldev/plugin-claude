---
name: pesquisa-juridica
description: >
  Skill para pesquisa jurídica em licitações e contratações públicas usando o
  MCP VectorGov. Use SEMPRE que o usuário fizer perguntas sobre legislação
  brasileira de licitações, compras públicas, contratos administrativos,
  Lei 14.133/2021, pregão, dispensa, inexigibilidade, ETP, termo de referência,
  registro de preços, habilitação, sanções ou qualquer tema de direito
  administrativo aplicado a contratações. Também use quando o usuário
  mencionar: "o que diz a lei sobre", "qual artigo", "norma", "decreto",
  "instrução normativa", "buscar na legislação", "consultar artigo",
  "quais normas", ou fizer perguntas genéricas como "pode me ajudar com
  licitação?" — o VectorGov tem as normas e esta skill sabe como buscá-las.
metadata:
  version: "0.1.0"
  author: "VectorGov"
---

# Pesquisa Jurídica com VectorGov

## O que é o VectorGov

O VectorGov é uma base curada, vigente e estruturada da legislação federal brasileira de licitações e contratações públicas. Diferente de buscar no Google ou no Planalto, ele entrega o texto legal correto (sem redações anteriores sobrepostas), com links de evidência verificáveis em cada resultado.

Há 6 ferramentas disponíveis. O trabalho central desta skill é escolher a certa para cada pergunta.

## Escopo da base — respeitar sempre

A base cobre a legislação federal de licitações e contratações públicas: Lei 14.133/2021 e normas correlatas (decretos, instruções normativas, portarias).

Perguntas de outros domínios jurídicos (trabalhista/CLT, tributário pessoal, civil, família, penal comum, previdência, consumidor, trânsito) NÃO são cobertas. Nesses casos, os trechos retornados serão apenas parecidos no vocabulário — "rescisão" na base é de contrato administrativo; "pena" é de crime em licitação. NUNCA adaptar esses dispositivos para responder a pergunta de outro domínio. Informar ao usuário que a pergunta está fora do escopo da base e dizer o que ela cobre.

## Mapa de decisão rápido

```
Pergunta do usuário
  │
  ├─ "O que diz o Art. X da Lei Y?" / referência exata a dispositivo
  │   └─► lookup
  │
  ├─ Pergunta temática ou aberta ("como funciona...", "quais as regras...")
  │   └─► hybrid  (busca padrão)
  │
  ├─ Sigla ou expressão literal ("onde aparece ETP?", "quem cita o art. 191?")
  │   └─► grep
  │
  ├─ Valor de tabela, código NCM, item de anexo de decreto/IN
  │   └─► consultar_tabela
  │
  ├─ "O que dá pra consultar?" / explorar por assunto
  │   └─► filtrar_por_tema  (sem parâmetro = catálogo de temas)
  │
  └─ Usuário perdido ou pedindo ajuda geral
      └─► menu
```

## As 6 Ferramentas — Quando Usar Cada Uma

### 1. `lookup` — Leitura Direta de Dispositivo

**Quando:** o usuário sabe EXATAMENTE qual dispositivo quer ler, ou é preciso confirmar o texto de um artigo citado em outra resposta.

**Sinais no pedido:** "Art. 75", "§ 1º do art. 6º", "inciso II do art. 75", "me mostra o artigo tal", "qual o texto do art. X".

**Como chamar:**

```
reference: "Art. 75 da Lei 14.133/2021"
```

**Formato da referência** — sempre montar de forma legível:

- `"Art. 75 da Lei 14.133/2021"`
- `"Art. 3, inciso II, IN 65/2021"`
- `"Art. 10 do Decreto 10.947/2022"`

**Batch:** aceita lista de até 20 referências de uma vez, com resolução paralela. Se o usuário pedir vários artigos, resolver todos em UMA chamada:

```
reference: ["Art. 75 da Lei 14.133", "Art. 3 da IN 65/2021", "Art. 1 do Decreto 10.947"]
```

**Parâmetros:** `include_siblings` e `include_parent` já vêm `true` por padrão (para um inciso, retorna também o caput e os demais incisos). Só desligar se o usuário quiser apenas o texto seco do dispositivo.

Se o usuário disser apenas "art. 75" sem especificar a norma: usar a norma que o contexto da conversa tornar óbvia (ex.: estavam falando da Lei 14.133). Se houver ambiguidade genuína, perguntar de qual norma.

### 2. `hybrid` — Busca Semântica (Padrão)

**Quando:** perguntas temáticas ou abertas, quando não se sabe qual artigo específico responde. É a busca PADRÃO da base.

**Sinais no pedido:** "quais as regras para...", "como funciona...", "o que a legislação diz sobre...", "qual o procedimento de...", dúvidas temáticas sem referência a artigo.

**Como chamar:**

```
query: "dispensa de licitação por valor"
top_k: 10          # padrão — não aumentar sem motivo
token_budget: 3500 # padrão
```

**Disciplina de `top_k` — importante:** manter o padrão 10 para responder perguntas. Janela maior NÃO melhora a resposta direta. Usar 20 SOMENTE quando o usuário pedir levantamento exaustivo ("todos os dispositivos sobre...", "varredura completa") ou em perguntas que claramente envolvem muitos artigos.

**Filtros opcionais:** `document_id` (ex.: `LEI-14.133-2021`, aceita com ou sem pontos), `tipo` (`LEI`, `DECRETO`, `IN`, `PORTARIA`), `ano`. Se o usuário citar a norma na própria pergunta, o filtro já é automático.

**Cartões de tabela:** se um resultado vier como cartão de tabela normativa (citação no formato "Tabela X do Anexo Y do..."), chamar `consultar_tabela` para obter o item exato. NUNCA responder valor de tabela a partir da amostra do cartão.

### 3. `grep` — Busca Textual Literal

**Quando:** o usuário quer encontrar onde um termo EXATO aparece no texto legal. Complementa a busca semântica — a semântica entende significado, o grep encontra a string literal.

**Sinais no pedido:** siglas (ETP, PCA, TR, SRP, ARP), referências cruzadas ("quais artigos citam o art. 191?"), expressões exatas ("menor preço", "dispensa eletrônica").

**Como chamar:**

```
query: "ETP"
document_id: "LEI-14133-2021"   # opcional; omitir = busca global
```

**Nota:** quando a busca literal não encontra nada, o backend cai automaticamente no índice curado (aliases e resumos) para resgatar o dispositivo — útil para siglas e sinônimos que não aparecem literalmente no texto.

### 4. `consultar_tabela` — Tabelas de Anexos

**Quando:** a pergunta envolve item exato de tabela normativa em anexo de decreto ou IN — alíquotas, taxas de depreciação, listas de bens, enquadramento por código NCM. Também sempre que `hybrid` retornar um cartão de tabela.

**Como chamar:**

```
chave: "8424.41.00"                 # código NCM, com ou sem pontos; tem precedência
termo: "turbina"                    # busca textual nas linhas, se não houver chave
document_id: "DECRETO-12.955-2026"  # opcional
anexo: "ANEXO-I"                    # opcional
vigente_em: "2025-06-30"            # opcional — consulta versões históricas por data
```

A consulta é determinística: retorna a linha exata com citação e link de evidência. A confirmação de que um item NÃO consta da lista (`nao_consta: true`) é informação jurídica válida — citar a ausência com a fonte.

### 5. `filtrar_por_tema` — Explorar por Assunto

**Quando:** o usuário quer navegar pelas normas por categorias curadas, ou descobrir o que existe na base sobre um assunto.

**Sinais no pedido:** "o que dá pra consultar?", "quero explorar sobre habilitação", "quais temas existem?".

**Como chamar:** sem parâmetro `theme`, retorna o catálogo de temas disponíveis. Com `theme` (slug, ex.: `licitacoes-14133`), retorna os dispositivos daquele tema, paginado. Se o usuário disser "próxima" ou "mais", chamar de novo com `page` incrementado e o mesmo `theme`.

### 6. `menu` — Ajuda Geral

**Quando:** o usuário parece perdido, pede ajuda ou quer saber o que é possível fazer. Exibir o menu retornado INTEGRALMENTE, sem resumir nem omitir seções.

## Regras de Resposta — SEMPRE Seguir

### 1. Entregar links de evidência

Cada resultado do VectorGov vem com dois links de verificação: `evidence_url` (trecho destacado na norma) e `document_url` (PDF original). SEMPRE incluir esses links ao lado de cada citação:

```
[1] Lei 14.133/2021, Art. 75 — [ver trecho](evidence_url) · [baixar PDF](document_url)
```

Esses links são a prova de que a informação é real. Copiar as URLs exatamente como recebidas — não modificar nem encurtar.

### 2. Estruturar a resposta

**Perguntas diretas** (ex.: "o que diz o art. 75?"): apresentar o texto do dispositivo, entregar os links de evidência e, se relevante, mencionar brevemente dispositivos relacionados.

**Perguntas temáticas** (ex.: "como funciona a dispensa por valor?"): começar com uma síntese em 2-3 frases, citar os dispositivos relevantes com seus textos-chave e entregar os links de cada dispositivo citado.

**Exploração** (ex.: "o que vocês têm?"): listar os temas ou normas de forma organizada e sugerir o que o usuário pode perguntar a seguir.

### 3. Combinar ferramentas quando a pergunta pedir

| Pergunta | Sequência |
|---|---|
| "Me mostra o art. 75 e o contexto das regras de dispensa" | `lookup` → `hybrid` |
| "Onde aparece 'ICTI' na Lei 14.133 e o que significa?" | `grep` → `hybrid` |
| "Pulverizadores NCM 8424.41.00 estão desonerados?" | `consultar_tabela` direto |
| "Quero explorar habilitação e depois ler os artigos principais" | `filtrar_por_tema` → `lookup` (batch) |

Não hesitar em chamar 2-3 ferramentas se a pergunta pedir. O usuário quer a resposta completa.

### 4. Lidar com ambiguidade

Não pedir vários esclarecimentos. Fazer a busca com a melhor interpretação e apresentar o resultado. Só perguntar quando houver ambiguidade genuína (ex.: "art. 75" sem norma e sem contexto). Se o resultado não for o que o usuário queria, ele reformula naturalmente.

### 5. Não inventar informação

Se a busca não retornar resultados relevantes, dizer isso. Não inventar artigos, números ou interpretações. O VectorGov existe para evitar alucinação — respeitar isso. Se a norma não estiver na base, informar e sugerir a consulta direta no Planalto (planalto.gov.br).

## Exemplos de Interação

**Exemplo 1 — Pergunta temática**
> "Quais são os casos de dispensa de licitação?"

Ação: `hybrid(query="casos de dispensa de licitação")`. Sintetizar os dispositivos retornados, citar o Art. 75 da Lei 14.133/2021 com incisos-chave, entregar links de evidência.

**Exemplo 2 — Consulta direta**
> "Me mostra o art. 6º da Lei 14.133"

Ação: `lookup(reference="Art. 6 da Lei 14.133/2021")`. Apresentar o texto com incisos e links.

**Exemplo 3 — Vários artigos de uma vez**
> "Preciso do art. 75 da 14.133, do art. 3 da IN 65 e do art. 1 do Decreto 10.947"

Ação: `lookup(reference=["Art. 75 da Lei 14.133/2021", "Art. 3 da IN 65/2021", "Art. 1 do Decreto 10.947/2022"])` — uma única chamada em batch.

**Exemplo 4 — Termo exato**
> "Onde aparece a sigla PCA na legislação?"

Ação: `grep(query="PCA")`. Listar os dispositivos onde aparece, com contexto e links.

**Exemplo 5 — Item de tabela**
> "A NCM 8424.41.00 está na lista de desoneração?"

Ação: `consultar_tabela(chave="8424.41.00")`. Responder com a linha exata e o link de evidência — ou citar a ausência com a fonte, se `nao_consta`.

**Exemplo 6 — Fora do escopo**
> "Quantos dias de férias a CLT garante?"

Ação: nenhuma busca. Informar que a base cobre legislação federal de licitações e contratações públicas, e que direito trabalhista está fora do escopo.

## Convenções de Referência

- **`document_id`** (filtros): `LEI-14.133-2021`, `DECRETO-10.947-2022`, `IN-65-2021` — aceita com ou sem pontos no número.
- **`reference`** (lookup): formato legível — `"Art. 75 da Lei 14.133/2021"`.

## Objetivo Final

O usuário não sabe (e não precisa saber) o nome das ferramentas. Ele pergunta em linguagem natural. O trabalho é: interpretar a intenção, escolher a ferramenta certa, chamar com os parâmetros certos e entregar resposta clara com texto legal e links verificáveis. A sensação deve ser de conversar com um especialista em legislação de contratações públicas — com acesso instantâneo à base normativa e prova de cada afirmação.
