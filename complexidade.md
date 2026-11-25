# Critérios de complexidade

O nível de complexidade de cada pergunta do CensoBench (campo `complexidade` em `censobench.json`) vem de uma
pontuação calculada sobre a SQL gold. A pontuação é a soma dos pontos de dez critérios, e define o nível:

| Nível | Pontuação |
|---|---|
| Fácil | 0 a 2 pontos |
| Média | 3 a 5 pontos |
| Difícil | 6 a 9 pontos |
| Muito Difícil | 10 pontos ou mais |

## Critérios

| Critério | Regra de pontuação | Máximo |
|---|---|---|
| *JOINs* | +1 por *JOIN*, com teto de 3 | 3 |
| *FULL* ou *CROSS JOIN* | +2 se a consulta contém `FULL OUTER JOIN` ou `CROSS JOIN` | 2 |
| CTEs | +2 pela presença de `WITH` e +1 por CTE além da primeira | – |
| *Window functions* | +3 se a consulta contém a cláusula `OVER()` | 3 |
| Subconsultas | número de blocos `SELECT` entre parênteses somado à profundidade máxima de aninhamento | – |
| Agregação aninhada | +2 se há agregação dentro de agregação | 2 |
| `HAVING` | +1 se a cláusula está presente | 1 |
| `NOT IN` ou `EXISTS` | +2 se há `NOT IN`, `EXISTS` ou `NOT EXISTS` | 2 |
| `CASE WHEN` | +1 se há expressão condicional | 1 |
| Mais de três tabelas | +1 se a consulta usa mais de três tabelas do esquema | 1 |

## Justificativa dos critérios

- **JOINs**: cada junção exige que o modelo identifique a tabela correta e a chave de ligação, o que amplia o espaço
  de *schema linking*. O teto de 3 pontos evita que consultas com muitas junções simples sejam superestimadas.
- **FULL ou CROSS JOIN**: são construções raras e de semântica mais exigente. O `FULL OUTER JOIN` exige o tratamento
  de valores nulos nas duas direções, e o `CROSS JOIN` exige atenção à cardinalidade do resultado.
- **CTEs**: cada *Common Table Expression* define um resultado intermediário que o modelo precisa manter ao escrever
  a consulta principal. Cada CTE adicional cria uma nova subtarefa e aumenta o tamanho da consulta esperada.
- **Window functions**: funções como `ROW_NUMBER()`, `LAG()` e `AVG() OVER()` exigem o uso correto de
  `PARTITION BY` e `ORDER BY` dentro de `OVER()`, ou seja, raciocinar sobre grupos e sobre a ordem das linhas ao
  mesmo tempo.
- **Subconsultas**: a pontuação combina duas dimensões, o número de blocos `SELECT` entre parênteses, incluindo os que
  definem CTEs, e a profundidade máxima de aninhamento. A profundidade considera apenas os parênteses que abrem um
  `SELECT`, e ignora os de funções como `COUNT()`, `EXTRACT()` e `COALESCE()`.
- **Agregação aninhada**: agregações dentro de agregações exigem que o modelo organize dois níveis de agrupamento na
  mesma expressão.
- **HAVING**: a cláusula filtra os grupos depois da agregação e exige que o modelo distinga esse filtro do `WHERE`,
  uma confusão recorrente em avaliações de *text-to-SQL*.
- **NOT IN ou EXISTS**: operadores de negação e de existência exigem raciocinar sobre o complemento de um conjunto,
  como nas perguntas sobre escolas ou turmas que deixaram de aparecer nos dados.
- **CASE WHEN**: expressões condicionais acrescentam lógica de decisão dentro da consulta.
- **Mais de três tabelas**: consultas que combinam muitas tabelas do esquema aumentam a chance de erro na escolha de
  atributos e de apelidos. As CTEs não são contadas como tabelas, pois são definidas na própria consulta.

## Exemplos

A tabela decompõe a pontuação de quatro perguntas do benchmark, uma de cada nível (o `id` é o de
`censobench.json`). Os critérios que não pontuam em nenhuma das quatro (*FULL* ou *CROSS JOIN*, agregação
aninhada, `NOT IN` ou `EXISTS` e mais de três tabelas) foram omitidos.

| Critério | Q77 | Q48 | Q11 | Q87 |
|---|---|---|---|---|
| *JOINs* | 0 | 1 | 1 | 1 |
| CTEs | 0 | 0 | 3 | 4 |
| *Window functions* | 0 | 0 | 0 | 3 |
| Subconsultas | 2 | 2 | 5 | 7 |
| `HAVING` | 0 | 0 | 0 | 1 |
| `CASE WHEN` | 0 | 0 | 0 | 1 |
| **Total** | **2** | **3** | **9** | **17** |
| **Nível** | Fácil | Média | Difícil | Muito Difícil |

- **Q77, Fácil**: não há junção nem CTE. A única subconsulta, `(SELECT MAX(ano) FROM escola)`, conta 1 bloco e
  profundidade 1, o que soma 2 pontos.
- **Q48, Média**: a junção entre `matricula` e `turma` soma 1 ponto, e a subconsulta do recorte temporal soma 2
  (1 bloco e profundidade 1), o que totaliza 3 pontos.
- **Q11, Difícil**: a autojunção `matriculas_escola m1 JOIN matriculas_escola m2` soma 1 ponto. As duas CTEs somam 3
  (2 pela presença e 1 pela segunda CTE). Há três blocos `SELECT` entre parênteses, os corpos das duas CTEs e o
  `MAX(ano)`, e o `MAX(ano)` está dentro da primeira CTE, o que dá profundidade 2 e soma 5 pontos. O total é 9.
- **Q87, Muito Difícil**: o `LEFT JOIN` soma 1 ponto, e as três CTEs somam 4 (2 pela presença e 1 por cada CTE
  adicional). O `ROW_NUMBER() OVER (...)` soma 3. Há cinco blocos `SELECT` entre parênteses, os corpos das três CTEs
  e os dois `MAX(ano)`, com profundidade 2, o que soma 7 pontos. A cláusula `HAVING` e o `CASE WHEN` somam 1 ponto
  cada. O total é 17.
