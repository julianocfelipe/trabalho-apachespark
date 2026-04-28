# Apache Spark (PySpark)

## O que é o Apache Spark?

O **Apache Spark** é um motor de processamento distribuído de código aberto. A principal vantagem em relação ao Hadoop MapReduce é o processamento **em memória**: em vez de gravar resultados intermediários em disco a cada etapa, o Spark mantém os dados na RAM, o que o torna significativamente mais rápido para cargas de trabalho iterativas e interativas.

---

## Principais Características

| Característica          | Descrição                                                            |
|------------------------|----------------------------------------------------------------------|
| **Processamento em memória** | Opera diretamente na RAM, eliminando I/O de disco desnecessário |
| **Multi-linguagem**    | Suporta Python (PySpark), Scala, Java, R e SQL                      |
| **Tolerância a falhas**| Recuperação automática via RDD Lineage                              |
| **Lazy Evaluation**    | Executa transformações apenas quando uma ação é solicitada          |
| **Distributed Computing** | Escala horizontalmente em clusters de centenas de nós           |

---

## Arquitetura

```
Driver Program
     │
     ▼
Spark Context ──────────── Cluster Manager
                                │
             ┌──────────────────┼──────────────────┐
             ▼                  ▼                  ▼
        Worker Node        Worker Node        Worker Node
        ┌─────────┐        ┌─────────┐        ┌─────────┐
        │Executor │        │Executor │        │Executor │
        │ Task  ││        │ Task  ││        │ Task  ││
        │ Task  ││        │ Task  ││        │ Task  ││
        └─────────┘        └─────────┘        └─────────┘
```

- **Driver**: Coordena a execução, mantém o SparkContext
- **Cluster Manager**: Gerencia recursos (YARN, Mesos, Kubernetes ou Standalone)
- **Executor**: Processo JVM que executa as tarefas nos workers

---

## Componentes do Ecossistema Spark

| Componente       | Finalidade                                      |
|------------------|-------------------------------------------------|
| **Spark SQL**    | Consultas SQL e DataFrames estruturados         |
| **Spark Streaming** | Processamento de streams em tempo real       |
| **MLlib**        | Machine Learning escalável                      |
| **GraphX**       | Processamento de grafos                         |
| **PySpark**      | API Python para o Spark                         |

---

## PySpark — API Python

O **PySpark** é a interface Python para o Apache Spark, permitindo escrever jobs Spark usando Python.

### Criando uma SparkSession

A `SparkSession` é o ponto de entrada para toda funcionalidade do Spark:

```python
from pyspark.sql import SparkSession

spark = SparkSession.builder \
    .appName("Minha Aplicação") \
    .config("spark.some.config", "valor") \
    .getOrCreate()
```

### DataFrames

O principal objeto de trabalho no PySpark é o **DataFrame** — uma coleção distribuída de dados organizada em colunas nomeadas (similar a uma tabela SQL):

```python
from pyspark.sql.types import StructType, StructField, StringType, IntegerType

schema = StructType([
    StructField("nome", StringType()),
    StructField("idade", IntegerType()),
])

data = [("Ana", 30), ("Bruno", 25)]
df = spark.createDataFrame(data, schema)
df.show()
```

**Saída:**
```
+-----+-----+
| nome|idade|
+-----+-----+
|  Ana|   30|
|Bruno|   25|
+-----+-----+
```

### Transformações e Ações

No Spark, operações são divididas em:

- **Transformações** (lazy): `filter()`, `select()`, `groupBy()`, `join()` — definem o plano de execução
- **Ações** (trigger): `show()`, `count()`, `collect()`, `write()` — disparam a execução real

```python
# Transformações (nada é executado ainda)
resultado = df.filter(df.idade > 25).select("nome")

# Ação (executa o plano)
resultado.show()
```

### Spark SQL

O PySpark permite usar SQL diretamente:

```python
df.createOrReplaceTempView("pessoas")

spark.sql("""
    SELECT nome, idade
    FROM pessoas
    WHERE idade > 25
    ORDER BY idade DESC
""").show()
```

---

## Configuração usada neste projeto

Neste projeto, o Spark é configurado em modo **local** (single machine), ideal para desenvolvimento e aprendizado:

```python
import os, sys

os.environ["PYSPARK_PYTHON"] = sys.executable
os.environ["PYSPARK_DRIVER_PYTHON"] = sys.executable
os.environ["HADOOP_HOME"] = "C:\\hadoop"

spark = SparkSession.builder \
    .appName("Loja Virtual") \
    .config("spark.sql.extensions", "...") \
    .getOrCreate()

spark.sparkContext.setLogLevel("ERROR")
```

---

## Por que usar o Spark para Data Engineering?

No contexto de Data Engineering, o Spark virou padrão porque resolve bem três problemas ao mesmo tempo: volume (escala horizontal em clusters), variedade (DataFrames estruturados, semi-estruturados e streams na mesma API) e velocidade (in-memory computing). Além disso, a integração nativa com Delta Lake, Iceberg, S3, HDFS e Kafka faz ele funcionar bem em arquiteturas de Data Lakehouse.

---

## Versões utilizadas

```
Apache Spark  : 3.5.0
PySpark       : 3.5.0
Python        : 3.11
Java          : 1.8 (JDK 8)
```
