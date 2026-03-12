# Análise de Variação do IPCA — Banco IP2CA

## Sobre 

Este projeto foi desenvolvido como solução para o case técnico do **Banco IP2CA**, um banco de varejo brasileiro com 30 anos de atuação e 50 milhões de clientes ativos.

O objetivo é automatizar a coleta, processamento e análise do **Índice de Preços ao Consumidor Amplo (IPCA)**, substituindo processos manuais lentos por uma solução moderna baseada em dados.

---

## 🛠️ Tecnologias Utilizadas

| Tecnologia | Finalidade |
|------------|------------|
| **Databricks ** | Processamento e armazenamento dos dados |
| **Python** | Coleta dos dados via API do Banco Central |
| **SQL** | Processamento, transformação e análise dos dados |
| **Power BI Desktop** | Visualização e dashboard |

---

## 📥 Fonte dos Dados

- **Instituição:** Banco Central do Brasil
- **Série:** 433 — Índice de Preços ao Consumidor Amplo (IPCA)
- **Período:** Janeiro de 1991 a Dezembro de 2025
- **URL:**  https://dadosabertos.bcb.gov.br/dataset/10844-indice-de-precos-ao-consumidor-amplo-ipca---servicos/resource/c0980df7-ad92-47af-b71c-790825f4710a

---

## ⚙️ Como Executar o Notebook

### Pré-requisitos
- Conta no [Databricks Community Edition](https://community.cloud.databricks.com)


### Passos
1. Faça o upload do arquivo `data/bcdata.sgs.10844.csv` no Databricks 
2. Importe o notebook `notebook/notebook-case-ipca.ipynb` 
3. Anexe o notebook ao cluster criado
4. Execute as células em ordem

### Tabelas geradas no Delta Lake

| Tabela | Descrição |
|--------|-----------|
| `ipca_mensal` | Dados mensais do IPCA tratados |
| `ipca_anual_delta` | IPCA acumulado por ano |
| `kpi_ipca` | Média do IPCA excluindo 2020 e 2021 |

---

## 🧮 Metodologia do Cálculo

O IPCA acumulado anual é calculado pelo **produto dos fatores mensais**, não pela soma simples dos percentuais. Isso é necessário porque a inflação de cada mês incide sobre o valor já corrigido do mês anterior (efeito composto).

**Exemplo:**
```
Janeiro: 10% → R$ 100,00 vira R$ 110,00
Fevereiro: 10% → R$ 110,00 vira R$ 121,00
Acumulado real: 21% (não 20%)
```

**Fórmula em SQL:**
```sql
ROUND((EXP(SUM(LN(1 + valor_mensal / 100))) - 1) * 100, 4) AS ipca_acumulado_anual
```

O `EXP(SUM(LN(...)))` é o recurso do SQL para realizar a multiplicação encadeada dos fatores mensais, equivalente a:
```
fator₁ × fator₂ × ... × fator₁₂
```

---

## 📊 Dashboard Power BI

O dashboard é dividido em duas seções históricas para melhor legibilidade:

### KPIs
| Indicador | Valor |
|-----------|-------|
| Média IPCA últimos 10 anos | 4,76% |
| Média IPCA últimos 5 anos (ex-pandemia) | 5,08% |
| Maior IPCA histórico | 2316,20% (1993) |
| Menor IPCA pós Plano Real | 1,22% (1998) |

### Gráficos
- **Período de Hiperinflação (1991–1994):** destaca o pico de 2316% em 1993 e o impacto do Plano Real
- **Pós Plano Real (1995–2025):** mostra a estabilização da inflação e as variações recentes

> Os anos de 2020 e 2021 foram excluídos do KPI de média por apresentarem comportamento atípico devido à pandemia de COVID-19.

---

## 💡 Insights para o Banco IP2CA

### 1. 📉 Estabilização pós Plano Real
A inflação caiu de **2316% em 1993** para **1,22% em 1998** após o Plano Real. Esse contexto histórico é essencial para o banco entender como produtos financeiros de longo prazo foram impactados ao longo do tempo.

**Ação recomendada:** Revisar contratos e produtos criados antes de 1995 que ainda estejam ativos e que possam conter cláusulas de reajuste desatualizadas.

---

### 2. 📈 IPCA médio recente acima da meta
A média dos últimos 5 anos excluindo a pandemia é de **5,08%**, acima da meta histórica do Banco Central de 4,5%. Isso indica um ambiente inflacionário persistente que impacta diretamente o custo de vida dos clientes.

**Ação recomendada:** Calibrar modelos de crédito e precificação de produtos financeiros considerando esse patamar como referência de inflação estrutural.

---

### 3. 🦠 Distorção pandêmica (2020–2021)
Os anos de 2020 e 2021 apresentaram comportamento atípico — queda inicial da demanda seguida de forte recomposição de preços. Usar esses anos como referência distorce qualquer modelo preditivo.

**Ação recomendada:** Excluir 2020 e 2021 de modelos preditivos e benchmarks de inflação, utilizando a média ajustada de 5,08% como base de planejamento.

---

### 4. 📅 Sazonalidade mensal do IPCA
Historicamente o IPCA apresenta picos em **janeiro** (reajustes de serviços como energia e transporte) e **julho** (reajustes de planos de saúde e educação).

**Ação recomendada:** Planejar campanhas de crédito e renegociação de dívidas para os meses de maior pressão inflacionária, antecipando o impacto no poder de compra dos clientes.

---

### 5. 💳 Correlação entre IPCA alto e inadimplência
Anos com IPCA acima de 8% — como 2015 (8,11%) e 2022 (7,57%) — tendem a coincidir com aumento da inadimplência, pois corroem o poder de compra das famílias de menor renda.

**Ação recomendada:** Monitorar o IPCA em tempo real como indicador antecedente de risco de crédito, ajustando limites e taxas de forma preventiva para os segmentos mais vulneráveis da base de clientes.

---

### 6. 🏦 Impacto na Selic e custo de captação
O Banco Central historicamente eleva a Selic em resposta a IPCAs acima da meta, o que aumenta o custo de captação dos bancos e pressiona as margens.

**Ação recomendada:** Desenvolver modelos de simulação que correlacionem IPCA com Selic para antecipar cenários de funding e precificação de produtos de crédito e investimento.

---

*Fonte dos dados: Banco Central do Brasil — Série 433 (IPCA)*
