# Semana 9 — Excel Avançado: Power Query

Mini-projeto da Capacitação em Ciência de Dados (Asimov Jr), dando continuidade à base de vendas de eletrônicos usada na Semana 8. O objetivo desta etapa foi sair do uso de fórmulas soltas e construir um fluxo de tratamento de dados completo, estruturado e reprodutível dentro do Excel, usando o Power Query como ferramenta central.

## Contexto

Peguei a mesma base de vendas de uma loja de eletrônicos (2023–2024) e simulei um cenário mais próximo da realidade de mercado: em vez de receber os dados já limpos, recebi uma exportação bruta de um sistema legado, com todos os problemas típicos desse tipo de fonte — cabeçalhos inconsistentes, texto desalinhado, linhas duplicadas, valores ausentes e erros de digitação. A partir daí, o projeto consistiu em montar um pipeline de ETL dentro do Power Query para transformar esse dado bruto em uma base confiável, e então construir indicadores, uma tabela dinâmica estruturada e um dashboard analítico em cima dela.

## Estrutura do arquivo

O entregável é um único arquivo Excel (`Semana9_PowerQuery_Dashboard.xlsx`) com 7 abas:

| Aba | Conteúdo |
|---|---|
| `Capa` | Visão geral do projeto e da estrutura do arquivo |
| `Base_Bruta` | Exportação bruta (434 linhas), com sujeira proposital: cabeçalhos inconsistentes, texto desalinhado, 14 duplicatas, 26 custos ausentes e 10 quantidades inválidas |
| `Power_Query_ETL` | Documentação passo a passo do tratamento aplicado no Power Query, incluindo o código M completo, pronto para colar no Editor Avançado |
| `Base_Tratada` | Resultado do "Fechar e Carregar" do Power Query — 411 linhas limpas, sem valores nulos, com métricas calculadas |
| `Medidas_Indicadores` | Indicadores construídos como medidas nomeadas (Gerenciador de Nomes) + consultas com múltiplos critérios |
| `Tabela_Dinamica` | Tabela dinâmica estruturada (Categoria x Canal e Região x Faixa de Receita), com totais e uma tabela auxiliar de tendência mensal |
| `Dashboard` | Painel executivo com cartões de indicador e 4 gráficos analíticos |

## O pipeline de tratamento (Power Query)

O tratamento aplicado à base bruta segue estas etapas, todas documentadas com o código M correspondente na aba `Power_Query_ETL`:

1. **Renomear colunas** — padroniza nomes inconsistentes (`" regiao"`, `"CANAL "`, `"QTD"` etc.) para um schema único.
2. **Remover colunas** — descarta a coluna `Observacao`, que não é usada na análise.
3. **Limpar e formatar texto** — `Text.Trim` + `Text.Clean` + `Text.Proper` em todas as colunas de texto, removendo espaços extras e uniformizando a capitalização.
4. **Corrigir tipos de dado** — converte data (texto → data), quantidade (→ inteiro) e valores monetários (→ decimal).
5. **Remover duplicatas** — elimina as 14 linhas 100% repetidas geradas por reenvio do sistema.
6. **Filtrar linhas inválidas** — remove os 10 registros com quantidade zero ou negativa (erro de digitação no ponto de venda).
7. **Agrupar por categoria** — calcula a razão média Custo/Preço de cada categoria, usando apenas as linhas com custo preenchido.
8. **Mesclar consultas (merge)** — junta essa razão média de volta à base principal pela chave Categoria.
9. **Preencher valores ausentes** — substitui os 26 custos nulos por `Preço Unitário × Razão Média da Categoria`, uma estimativa mais fiel do que usar uma média geral ou um valor fixo.
10. **Colunas calculadas** — adiciona Receita, Custo Total, Lucro e Margem %.
11. **Coluna condicional** — classifica cada venda em faixa de receita (Alta / Média / Baixa).
12. **Colunas de data** — extrai Ano e Mês para permitir análises temporais.
13. **Ordenar e carregar** — ordena por data crescente e carrega o resultado como tabela na aba `Base_Tratada`.

O resultado final: **434 linhas brutas → 411 linhas tratadas**, zero valores nulos.

## Indicadores e medidas

Em vez de espalhar fórmulas soltas pela planilha, os indicadores principais foram implementados como **medidas nomeadas** no Gerenciador de Nomes do Excel, permitindo que qualquer fórmula da pasta de trabalho as referencie diretamente pelo nome:

- `Medida_ReceitaTotal` — R$ 1.015.597,61
- `Medida_LucroTotal` — R$ 340.779,86
- `Medida_MargemMedia` — 33,4%
- `Medida_TicketMedio` — R$ 2.471,04
- `Medida_TotalPedidos` — 411
- `Medida_QtdTotal` — 513

Além disso, a aba `Medidas_Indicadores` traz uma consulta com **três critérios simultâneos** (Categoria + Canal + Ano, via SOMASES/CONT.SES) e uma busca mais avançada, combinando ÍNDICE + CORRESP + SOMARPRODUTO, para localizar automaticamente o cliente responsável pela maior venda dentro de dois critérios escolhidos.

## Tabela dinâmica estruturada

A aba `Tabela_Dinamica` cruza os dados em duas dimensões:

- **Receita por Categoria x Canal**, com totais de linha e coluna. Principais achados: o canal **Online** supera a **Loja Física** em receita (R$ 586.455,89 contra R$ 429.141,72), e **Smartphones** é a categoria líder (R$ 468.675,37).
- **Número de pedidos por Região x Faixa de Receita**, mostrando que a região **Sudeste** concentra o maior volume de pedidos.

Uma terceira tabela auxiliar agrega a receita mês a mês (2023–2024) e alimenta o gráfico de tendência do dashboard.

Todas as tabelas são construídas com fórmulas (SOMASES/CONT.SES) em vez de um pivot cache tradicional, o que garante que os números recalculam automaticamente junto com a base tratada, sem depender de um "Atualizar" manual.

## Dashboard

A aba `Dashboard` reúne 4 cartões de indicador (ligados diretamente às medidas nomeadas) e 4 gráficos: receita por categoria, participação por canal, evolução mensal da receita e pedidos por região e faixa de receita — um painel pensado para leitura executiva rápida.

## Como reproduzir o tratamento no Power Query

1. Abra o arquivo no Excel e selecione a tabela `tblBruta` na aba `Base_Bruta`.
2. Vá em **Dados > Obter Dados > De Tabela/Intervalo**.
3. No Editor do Power Query, abra o **Editor Avançado** e cole o código M disponível na aba `Power_Query_ETL`.
4. Acompanhe o painel de etapas aplicadas — cada passo corresponde a um item da tabela documentada na mesma aba.
5. Finalize com **Fechar e Carregar Como Tabela**.

## Entregas

- Arquivo Excel: `Semana9_PowerQuery_Dashboard.xlsx`
- Repositório GitHub (usuário **Mazzer**) com este README e o link da pasta no Drive
- Apresentação ao vivo da situação-problema, reproduzindo o tratamento no Power Query em tempo real
