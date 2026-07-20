# VectorGov para Claude

Pesquisa jurídica em licitações e contratos públicos, direto no Claude. A base é curada e vigente. Cada resposta traz link de evidência para conferir na fonte original.

## O que o plugin faz

Conecta o Claude à base do VectorGov e ensina o Claude a pesquisar nela do jeito certo. Você pergunta em linguagem natural. O Claude escolhe a ferramenta adequada, encontra o dispositivo legal ou o acórdão pertinente e responde com o texto correto e a fonte verificável.

Exemplos de perguntas:

- "Quais são os casos de dispensa de licitação?"
- "Me mostra o art. 75 da Lei 14.133"
- "O que o TCU entende sobre exigência de atestado de capacidade técnica?"
- "Qual o código CATMAT para comprar cadeiras giratórias?"

## Componentes

- **Skill `pesquisa-juridica`** — a metodologia de pesquisa: qual ferramenta usar para cada tipo de pergunta, como montar referências e como citar com evidência.
- **Conexão MCP** — servidor do VectorGov em `https://mcp.vectorgov.io/mcp`, com ferramentas de consulta em três frentes: **legislação** (busca semântica, leitura de artigo, busca literal, navegação por tema, menu), **jurisprudência do TCU** (buscar, localizar termo, navegar e ler acórdãos) e **catálogo de compras** CATMAT/CATSER.

## Como começar

1. Instale o plugin.
2. Faça uma pergunta sobre licitação ou contratos públicos.
3. Na primeira consulta, o Claude vai pedir para você entrar na sua conta VectorGov (login OAuth). Crie sua conta em [vectorgov.io](https://vectorgov.io).

Não precisa configurar chave de API nem variável de ambiente. O login OAuth resolve tudo.

## O que a base cobre

Legislação federal de licitações e contratações públicas (Lei 14.133/2021 e normas correlatas — decretos, instruções normativas, portarias), a jurisprudência do TCU sobre o tema (recorte do Informativo de Licitações e Contratos) e o catálogo oficial de compras (CATMAT/CATSER). Perguntas de outros domínios jurídicos (trabalhista, tributário pessoal, civil, penal comum) estão fora do escopo — e o Claude vai te avisar em vez de improvisar.

## Por que isso importa

LLMs alucinam artigo de lei. Citam dispositivo revogado, inventam número de inciso, misturam redações. O VectorGov corta isso na raiz: o texto vem da base curada e vigente, e cada citação chega com link para o trecho destacado na norma e para o PDF original. Você confere a fonte em um clique.

## Suporte

Dúvidas e suporte: [vectorgov.io](https://vectorgov.io)
