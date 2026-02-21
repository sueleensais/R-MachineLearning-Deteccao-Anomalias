# Detecção de Anomalias em Transações Financeiras (R + Power BI)

Projeto desenvolvido durante o capítulo 18 do curso **"Microsoft Power BI para Business Intelligence e Data Science"** da Data Science Academy.  
Os dados utilizados são **fictícios**.

## Descrição

Este projeto simula um cenário em que uma empresa financeira possui dados históricos de clientes com duas transações (chamadas de `transacao1` e `transacao2`).  
Os gestores suspeitam que algumas dessas transações possam ser fraudulentas e desejam identificar **anomalias**.  

Utilizando **Machine Learning com o algoritmo Isolation Forest**, os dados foram agrupados e analisados para detectar possíveis anomalias.  
O resultado foi disponibilizado em **gráficos no Power BI**, permitindo uma visualização clara e interativa.


## Definição do Problema

- Detectar transações financeiras que possam ser consideradas **anomalias**.  
- Criar um modelo de Machine Learning em R para identificar padrões e destacar registros suspeitos.  
- Disponibilizar os resultados em **dashboards no Power BI** para análise visual.  


## Dados

- `dados_historicos.csv`: dataset fictício com transações financeiras históricas.  
- `novos_dados.csv`: dataset fictício com novas transações para validação do modelo.  
- `previsoes_novos_dados.csv`: resultado das previsões, incluindo o **anomaly score** e o status (`anomalia` ou `normal`).  


## Ferramentas Utilizadas

- **R** (tidyverse, dplyr, solitude, ggplot2, readr)  
- **Rtools**  
- **RStudio**  
- **Power BI Desktop**  

## Passo a Passo

1. **Instalação e carregamento de pacotes R**  
```r
# Instala os pacotes
install.packages("tidyverse")
install.packages("dplyr")
install.packages("solitude")
install.packages("ggplot2")
install.packages("readr")

# Carrega os pacotes nesta sessão R
library(tidyverse)
library(dplyr)
library(solitude)
library(ggplot2)
library(readr)
```

2. **Carregamento e visualização dos dados históricos**
```r
# Carrega os dados históricos
dados_historicos_dsa <- read_csv("dados_historicos.csv")

# Visualiza os dados históricos
View(dados_historicos_dsa)
```

3. **Construção Modelo de Machine Learning**
```r
# Consulta Documentação do Algoritmo
?isolationForest 

# Cria o modelo de Machine Learning com algoritmo isolationForest
modelo_ml_dsa = isolationForest$new() 

# Treina o modelo
modelo_ml_dsa$fit(dados_historicos_dsa)

# Faz as previsões com o modelo usando os dados históricos
previsoes_historico = dados_historicos_dsa %>%
  modelo_ml_dsa$predict() %>%
  arrange(desc(anomaly_score))
```
4. **Fazendo Previsões com o Modelo de Detecção de Anomalias**
```r
# Faz as previsões com o modelo usando os dados históricos
previsoes_historico = dados_historicos_dsa %>%
  modelo_ml_dsa$predict() %>%
  arrange(desc(anomaly_score))

# Visualiza as previsões
View(previsoes_historico)

# Density Plot 
plot(density(previsoes_historico$anomaly_score))

# Quanto maior o anomaly score maior a chance do registro ser uma anomalia
```
5. **Definindo o Score de Anomalia**
```r
# Define como regra que anomaly score acima de 0.62 é uma anomalia
indices_historico = previsoes_historico[which(previsoes_historico$anomaly_score > 0.62)]

# Faz o filtro
anomalias_historico = dados_historicos_dsa[indices_historico$id, ]
normais_historico = dados_historicos_dsa[-indices_historico$id, ]

# Gráfico
colors()
ggplot() + 
  geom_point(data = normais_historico, 
             mapping = aes(transacao1,transacao2), 
             col = "skyblue3", 
             alpha = 0.5) + 
  geom_point(data = anomalias_historico,
             mapping = aes(transacao1,transacao2), 
             col = "red2", 
             alpha = 0.8)
```
6. **Aplicando o Modelo de Detecção de Anomalias a Novos Dados**
```r
# Carregando e visualizando novos dados
novos_dados_dsa <- read.csv("novos_dados.csv")
View(novos_dados_dsa)

# Previsões com o modelo treinado
previsoes_novos_dados = modelo_ml_dsa$predict(novos_dados_dsa)

# Se o anomaly score é maior que 0.62 consideramos como anomalia
indices_novos_dados = previsoes_novos_dados[which(previsoes_novos_dados$anomaly_score > 0.62)]

# Filtro
anomalias_novos_dados = novos_dados_dsa[indices_novos_dados$id, ]
normais_novos_dados = novos_dados_dsa[-indices_novos_dados$id, ]

# Gráfico
ggplot() + 
  geom_point(data = normais_novos_dados, 
             mapping = aes(transacao1,transacao2), 
             col = "turquoise3", 
             alpha = 0.5) + 
  geom_point(data = anomalias_novos_dados, 
             mapping = aes(transacao1,transacao2), 
             col = "tomato3", 
             alpha = 0.8)

# Visualizando previsões
View(previsoes_novos_dados)
```
7. **Analisando as Anomalias com Box Plot em Linguagem R**
```r
# Arredondando a coluna 'anomaly_score' para 2 casas decimais
previsoes_novos_dados <- previsoes_novos_dados %>%
  mutate(anomaly_score = round(anomaly_score, 2))
 
# Criando uma nova coluna (status) com base na condição score de anomalias
previsoes_novos_dados <- previsoes_novos_dados %>%
  mutate(status = ifelse(anomaly_score > 0.62, "anomalia", "normal"))
  
# Criando o box plot
ggplot(previsoes_novos_dados, aes(x = status, y = anomaly_score, fill = status)) +
  geom_boxplot() +
  labs(title = "Box Plot de Anomalias e Normais",
       x = "Status",
       y = "Anomaly Score") +
  theme_minimal() +
  scale_fill_manual(values = c("anomalia" = "red", "normal" = "blue")) +
  theme(legend.position = "none")

# Salvando em disco
write.csv(previsoes_novos_dados, "previsoes_novos_dados.csv")
```
8. **Analisando as Anomalias com Box Plot no Power BI**

- Passo 1: Carregar o arquivo previsoes_novos_dados.csv
- Passo 2: Transformar dados da coluna “anomaly_score” para números decimais.
- Passo 3: Criar script R para gerar o gráfico Box Plot

```r
# O código a seguir para criar um dataframe e remover as linhas duplicadas sempre é executado e age como um preâmbulo para o script: 

# dataset <- data.frame(status, anomaly_score)
# dataset <- unique(dataset)

# Cole ou digite aqui seu código de script:

library(ggplot2)

# Criando o box plot
ggplot(dataset, aes(x = status, y = anomaly_score, fill = status)) +
  geom_boxplot() +
  labs(title = "",
       x = "Status",
       y = "Anomaly Score") +
  theme_minimal() +
  scale_fill_manual(values = c("anomalia" = "red", "normal" = "blue")) +
  theme(legend.position = "none")
```

## **Dashboard Final**
![Dashboard Power BI](dashboard.png)

## **Conclusão**
Este projeto demonstra como R e Power BI podem ser integrados para resolver problemas de detecção de anomalias em dados financeiros, oferecendo insights visuais e práticos para apoiar a tomada de decisão.

