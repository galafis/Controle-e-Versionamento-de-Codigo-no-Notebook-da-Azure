# Controle e Versionamento de Código no Notebook da Azure

Este projeto, proposto pela **DIO** e feito por mim, demonstra como utilizar o Azure Databricks para versionamento e organização de notebooks em ambientes de dados. A proposta inclui a criação de clusters, importação de arquivos, execução de notebooks com auxílio de inteligência artificial, além da integração com Azure DevOps para controle de código e automação de esteiras de CI/CD. É apresentado o uso prático da IA integrada ao Databricks para geração de código em Python e Spark, facilitando a criação de notebooks interativos com filtros, sumarizações, visualizações e comentários explicativos. Também são exploradas boas práticas de organização, exportação e reaproveitamento de notebooks, bem como o uso de recursos do Microsoft Learn, que oferecem exercícios guiados e roteiros de aprendizado. A abordagem permite trabalhar de forma colaborativa, segura e com versionamento estruturado em ambientes de análise, engenharia de dados e machine learning dentro da plataforma Azure.

## Insights e Possibilidades

### Integração com DevOps
A integração do Azure Databricks com o Azure DevOps proporciona um fluxo de trabalho completo para desenvolvimento, teste e implantação de notebooks. Isso permite:

- Controle de versão eficiente dos notebooks
- Automação de testes e validação de código
- Implantação contínua em ambientes de produção
- Rastreabilidade de alterações e colaboração entre equipes

### Uso de IA para Desenvolvimento
A integração de IA no Databricks revoluciona o desenvolvimento de notebooks:

```python
# Exemplo de uso de IA para gerar código Spark
# Basta descrever o que você precisa em um comentário:

# Gerar código para ler dados CSV, filtrar registros com valores nulos e calcular estatísticas básicas
from pyspark.sql.functions import col, count, when, isnan, avg, min, max

# O código abaixo seria sugerido pela IA
df = spark.read.format("csv").option("header", "true").load("/path/to/data.csv")
df_filtered = df.filter(~col("column_name").isNull())
df_stats = df_filtered.select(
    count("*").alias("total_records"),
    avg("numeric_column").alias("average_value"),
    min("numeric_column").alias("min_value"),
    max("numeric_column").alias("max_value")
)
display(df_stats)
```

### Organização e Reaproveitamento
A estruturação adequada de notebooks permite:

- Criação de bibliotecas de funções reutilizáveis
- Modularização de código para manutenção simplificada
- Compartilhamento de componentes entre projetos
- Documentação integrada e autoexplicativa

## Processo de Trabalho com Azure Databricks

### 1. Criação e Configuração de Clusters

O processo começa com a criação de clusters otimizados para as necessidades específicas do projeto:

```python
# Configuração programática de cluster via API
from databricks.sdk import WorkspaceClient
from databricks.sdk.service import compute

w = WorkspaceClient()

cluster_config = compute.ClusterSpec(
    cluster_name="MeuClusterDatabricks",
    spark_version="11.3.x-scala2.12",
    node_type_id="Standard_DS3_v2",
    autoscale=compute.AutoScale(min_workers=1, max_workers=5),
    spark_conf={
        "spark.databricks.delta.preview.enabled": "true",
        "spark.databricks.io.cache.enabled": "true"
    },
    autotermination_minutes=60
)

cluster = w.clusters.create(cluster_config)
print(f"Cluster criado com ID: {cluster.cluster_id}")
```

### 2. Importação e Organização de Arquivos

A importação de arquivos pode ser feita via interface ou programaticamente:

```python
# Importação de arquivos para o DBFS
dbutils.fs.cp("file:/local/path/arquivo.csv", "dbfs:/FileStore/datasets/arquivo.csv")

# Listagem de arquivos importados
files = dbutils.fs.ls("dbfs:/FileStore/datasets/")
display(files)
```

### 3. Versionamento com Git e Azure DevOps

Integração com repositórios Git para controle de versão:

```bash
# Comandos executados no terminal do Databricks
git init
git remote add origin https://dev.azure.com/minha-org/meu-projeto/_git/databricks-notebooks
git add .
git commit -m "Versão inicial dos notebooks"
git push -u origin master
```

No notebook Databricks:
```python
# Verificação do status do repositório
%sh
git status
git log --oneline -5
```

### 4. Automação de CI/CD com Azure DevOps

Exemplo de pipeline YAML para automação:

```yaml
# azure-pipelines.yml
trigger:
- main

pool:
  vmImage: 'ubuntu-latest'

steps:
- task: UsePythonVersion@0
  inputs:
    versionSpec: '3.8'
    addToPath: true

- script: |
    pip install databricks-cli pytest
    databricks configure --token
    echo $(DATABRICKS_TOKEN) | databricks configure --token
  displayName: 'Configurar CLI do Databricks'

- script: |
    databricks workspace import_dir ./notebooks /Shared/Projeto
  displayName: 'Implantar notebooks'

- script: |
    pytest ./tests
  displayName: 'Executar testes'
```

### 5. Execução de Notebooks com Parâmetros

Execução parametrizada para flexibilidade:

```python
# Definição de widgets para parâmetros
dbutils.widgets.text("data_inicio", "2023-01-01", "Data Início")
dbutils.widgets.text("data_fim", "2023-12-31", "Data Fim")

# Recuperação dos parâmetros
data_inicio = dbutils.widgets.get("data_inicio")
data_fim = dbutils.widgets.get("data_fim")

# Uso dos parâmetros na consulta
df = spark.sql(f"""
SELECT *
FROM tabela_dados
WHERE data BETWEEN '{data_inicio}' AND '{data_fim}'
""")

display(df)
```

### 6. Visualizações e Análises Interativas

Criação de visualizações avançadas:

```python
# Importação de bibliotecas para visualização
import matplotlib.pyplot as plt
import seaborn as sns

# Preparação dos dados
resultados = spark.sql("""
SELECT categoria, SUM(valor) as total
FROM vendas
GROUP BY categoria
ORDER BY total DESC
LIMIT 10
""").toPandas()

# Criação de visualização
plt.figure(figsize=(10, 6))
sns.barplot(x='categoria', y='total', data=resultados)
plt.title('Total de Vendas por Categoria')
plt.xticks(rotation=45)
plt.tight_layout()
display()
```

## Recursos de Aprendizado

O Microsoft Learn oferece recursos valiosos para aprofundar o conhecimento em Azure Databricks:

- [Introdução ao Azure Databricks](https://learn.microsoft.com/pt-br/training/modules/intro-to-azure-databricks/)
- [Engenharia de Dados com Azure Databricks](https://learn.microsoft.com/pt-br/training/paths/data-engineer-azure-databricks/)
- [Ciência de Dados com Azure Databricks](https://learn.microsoft.com/pt-br/training/paths/perform-data-science-azure-databricks/)

## Conclusão

O Azure Databricks, quando combinado com práticas adequadas de versionamento e organização de código, proporciona um ambiente robusto e colaborativo para projetos de dados. A integração com Azure DevOps e o uso de IA para assistência no desenvolvimento elevam a produtividade e a qualidade dos notebooks, permitindo que equipes trabalhem de forma mais eficiente em projetos complexos de análise, engenharia de dados e machine learning.


## 📋 Descrição

Descreva aqui o conteúdo desta seção.


## 📦 Instalação

Descreva aqui o conteúdo desta seção.


## 💻 Uso

Descreva aqui o conteúdo desta seção.


## 📄 Licença

Descreva aqui o conteúdo desta seção.
