# CensoBench

Benchmark de Text-to-SQL em português para o domínio de políticas públicas educacionais, construído sobre o
Censo Escolar Brasileiro (INEP, via Base dos Dados). São 107 perguntas em linguagem natural, cada uma com a
consulta SQL de referência (gold) e o nível de complexidade.

## Arquivo

`censobench.json` é uma lista com as 107 perguntas. Cada item tem:

```json
{
  "id": 1,
  "pergunta_nl": "Qual o número de matrículas ao longo dos últimos 3 anos por estado, rede de ensino e tipo de ensino?",
  "sql": "SELECT ...",
  "complexidade": "Média"
}
```

- `id`: número da pergunta (1 a 107).
- `pergunta_nl`: a pergunta, como um usuário a escreveria.
- `sql`: a consulta de referência (PostgreSQL).
- `complexidade`: Fácil, Média, Difícil ou Muito Difícil. O cálculo está em [complexidade.md](complexidade.md).

| Complexidade | Perguntas |
|---|---|
| Fácil | 46 |
| Média | 29 |
| Difícil | 8 |
| Muito Difícil | 24 |

## Léxico

`censo_lexicon.json` é o léxico do domínio usado para normalizar as perguntas na análise linguística: as regras
(padrão regex e substituição) de cada nível de normalização.

## Dados

As consultas rodam sobre as tabelas do Censo Escolar disponibilizadas pela
[Base dos Dados](https://basedosdados.org/) (`br_inep_censo_escolar`), carregadas em PostgreSQL.
