# Para onde foi o dinheiro das campanhas municipais no Ceará em 2024?

Análise exploratória das despesas de campanha declaradas ao TSE nas Eleições Municipais de 2024,
recorte do estado do Ceará.

**Stack:** Python · pandas · NumPy · Matplotlib · Seaborn
**Fonte:** [Repositório de Dados Eleitorais do TSE](https://dadosabertos.tse.jus.br/) — dados abertos
**Volume:** 144.613 registros · 53 colunas · R$ 259,3 milhões · 184 municípios · 9.949 candidatos

---

## Pergunta que guia a análise

> Para onde foi o dinheiro das campanhas municipais cearenses de 2024, e quão concentrado ele está?

Três eixos de concentração investigados: **geográfica** (quais municípios), **entre candidatos**
(quem gasta) e **entre fornecedores** (quem recebe).

---

## Principais achados

| Achado | Número |
|---|---|
| Gasto total declarado | **R$ 259,3 mi** em 141.689 itens |
| Participação de Fortaleza | **40,7%** — de 184 municípios |
| Top 1% das despesas | **38,5%** do valor total |
| Top 1% dos fornecedores | **56,5%** do valor total |
| Gini do gasto (dentro de cada cargo) | **~0,70** |
| Maior fornecedor do estado | **Meta/Facebook — R$ 16,1 mi, 717 candidatos, 27 partidos** |
| Gasto digital | 9,1% do total, **81% concentrado em candidaturas a prefeito** |

O maior fornecedor de campanha do Ceará não é uma gráfica nem uma agência local — é uma big tech.
Esse resultado **só aparece depois da consolidação por CNPJ**: o Facebook estava fragmentado em
86 grafias diferentes no campo de texto livre `NM_FORNECEDOR`.

---

## Problemas de qualidade encontrados e tratados

Boa parte do notebook é sobre descobrir e resolver problemas que não são visíveis à primeira vista:

1. **`SQ_DESPESA` não é chave primária.** Cada linha é um *item* dentro de um documento fiscal
   (um mesmo CNPJ/nota gera até 120 linhas). Um `drop_duplicates()` reflexo apagaria **R$ 45 milhões**
   de itens legítimos. Hipótese validada checando `DS_DESPESA` e `NR_DOCUMENTO` dentro de cada grupo.

2. **Valores sentinela documentados.** O leiame do TSE define `#NULO`/`-1`, `#NE`/`-3` e
   `NÃO DIVULGÁVEL`/`-4`. Um `isna()` não detecta nenhum deles. A auditoria distingue três situações
   diferentes: anonimização deliberada (CPF do candidato, 100% mascarado), esparsidade estrutural
   (campos que só se preenchem em casos raros) e ausência explicada por regra de negócio
   (CNAE só existe para PJ).

3. **CPF/CNPJ lido como inteiro perde zeros à esquerda.** Identificadores foram normalizados
   para 11 (CPF) ou 14 (CNPJ) dígitos.

4. **Nome de fornecedor é texto livre.** 47.659 nomes distintos para 43.395 documentos reais.
   O agrupamento correto é pelo CPF/CNPJ, com `NM_FORNECEDOR_RFB` apenas como rótulo.

5. **Encoding Latin-1, separador `;`, decimal brasileiro.** Um `read_csv` com defaults quebra o arquivo.

6. **Datas impossíveis.** 241 despesas datadas de 2025/2026 — retificações posteriores à eleição,
   sinalizadas e excluídas da série temporal.


## Decisões analíticas que vale destacar

- **Mediana em vez de média.** A distribuição tem assimetria de 148 e a média (R$ 1.830) é 3,7x a
  mediana (R$ 500). A média não descreve o gasto típico.
- **Gini separado por cargo.** O Gini geral de 0,862 é parcialmente artificial: mistura prefeitos e
  vereadores, populações de escalas distintas. Isolando cada cargo, cai para ~0,70 — ainda alto,
  mas agora interpretável.
- **Outliers mantidos.** As campanhas milionárias de Fortaleza não são erro de digitação: são o
  fenômeno mais interessante da base. Removê-las como "outlier estatístico" apagaria a história.
- **Agrupamento por `SQ_CANDIDATO`, não por nome.** Homônimos entre municípios quebrariam a contagem.
