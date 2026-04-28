# Apache Iceberg

## O que é o Apache Iceberg?

O **Apache Iceberg** é um formato de tabela aberta criado originalmente pela Netflix e depois doado para a Apache Software Foundation. O principal diferencial dele é tratar uma tabela como um objeto com metadados próprios (schema, partições, histórico de versões), ao invés de apenas uma pasta de arquivos Parquet. Isso resolve vários problemas de escalabilidade e consistência dos Data Lakes tradicionais.

---

## Principais Características

| Característica             | Descrição                                                               |
|---------------------------|-------------------------------------------------------------------------|
| **ACID Transactions**     | Operações atômicas e isoladas, seguras para leituras e escritas concorrentes |
| **Hidden Partitioning**   | Particionamento transparente — queries não precisam especificar partições |
| **Partition Evolution**   | Muda estratégias de particionamento sem migração de dados               |
| **Multi-engine**          | Compatível com Spark, Flink, Trino, Hive, Presto, Dremio               |
| **Row-level Operations**  | UPDATE e DELETE nativos                                                 |

---

## Arquitetura

O Iceberg separa os **metadados** dos **dados**, usando uma hierarquia de três camadas:

```
Catalog (referência ao current metadata)
    │
    ▼
Metadata File (metadata/v3.metadata.json)
    ├── Schema
    ├── Partition Spec
    └── Snapshots
            │
            ▼
    Manifest List (snap-xxx-1.avro)
            │
            ▼
    Manifest Files (xxx-m0.avro)
            │
            ▼
    Data Files (data/*.parquet)
```

### Tipos de Catálogos

| Tipo        | Uso                                       |
|------------|-------------------------------------------|
| **Hadoop** | Sistema de arquivos local/HDFS (dev/test) |
| **Hive**   | Integração com metastore Hive             |
| **REST**   | Catálogo via API REST (cloud-native)      |
| **Glue**   | AWS Glue (Amazon)                         |
| **Nessie**  | Controle de versão Git-like para tabelas |

---

## Configuração com PySpark

### Instalação (uv)

```bash
uv add pyspark==3.5.0
# O Iceberg é carregado via spark.jars.packages
```

### Inicializando a SparkSession com Iceberg

```python
from pyspark.sql import SparkSession
import os

ICEBERG_JAR = "org.apache.iceberg:iceberg-spark-runtime-3.5_2.12:1.4.3"
WAREHOUSE = os.path.abspath("./iceberg-warehouse")

spark = SparkSession.builder \
    .appName("Iceberg Demo") \
    .config("spark.jars.packages", ICEBERG_JAR) \
    .config("spark.sql.extensions",
            "org.apache.iceberg.spark.extensions.IcebergSparkSessionExtensions") \
    .config("spark.sql.catalog.local",
            "org.apache.iceberg.spark.SparkCatalog") \
    .config("spark.sql.catalog.local.type", "hadoop") \
    .config("spark.sql.catalog.local.warehouse", WAREHOUSE) \
    .getOrCreate()
```

---

## Operações DML

### DDL — Criar Namespace e Tabela

```python
spark.sql("CREATE NAMESPACE IF NOT EXISTS local.loja")

spark.sql("""
    CREATE TABLE IF NOT EXISTS local.loja.produtos (
        id        INT,
        nome      STRING,
        categoria STRING,
        preco     DOUBLE,
        estoque   INT
    ) USING iceberg
""")
```

### INSERT — Inserir Dados

```python
spark.sql("""
    INSERT INTO local.loja.produtos VALUES
        (1, 'Notebook Dell',   'Eletrônicos', 3500.0, 50),
        (2, 'Mouse Logitech',  'Periféricos',   80.0, 200),
        (3, 'Teclado Mecânico','Periféricos',  150.0, 120)
""")
```

**Resultado:**
```
+---+----------------+-----------+------+-------+
| id|            nome|  categoria| preco|estoque|
+---+----------------+-----------+------+-------+
|  1|   Notebook Dell|Eletrônicos|3500.0|     50|
|  2|  Mouse Logitech| Periféricos|  80.0|    200|
|  3|Teclado Mecânico| Periféricos| 150.0|    120|
+---+----------------+-----------+------+-------+
```

### UPDATE — Atualizar Dados

```python
spark.sql("""
    UPDATE local.loja.produtos
    SET preco = preco * 1.10
    WHERE categoria = 'Eletrônicos'
""")
```

**Resultado:**
```
+---+-------------+-----------+------+-------+
| id|         nome|  categoria| preco|estoque|
+---+-------------+-----------+------+-------+
|  1| Notebook Dell|Eletrônicos|3850.0|     50|
|  4|   Monitor 24"|Eletrônicos|1320.0|     75|
+---+-------------+-----------+------+-------+
```

### DELETE — Remover Dados

Assim como no Delta Lake, o Iceberg também suporta DELETE nativo. No exemplo abaixo removemos produtos sem estoque:

```python
spark.sql("""
    DELETE FROM local.loja.produtos
    WHERE estoque = 0
""")
```

O Iceberg registra essa operação como um novo snapshot. Os arquivos Parquet anteriores não são apagados imediatamente — o novo snapshot simplesmente não os referencia mais.

---

## Iceberg vs. Delta Lake

| Funcionalidade          | Delta Lake        | Apache Iceberg    |
|------------------------|-------------------|-------------------|
| ACID Transactions      | ✅                | ✅                |
| UPDATE / DELETE        | ✅                | ✅                |
| Partition Evolution    | ❌                | ✅                |
| Hidden Partitioning    | ❌                | ✅                |
| Multi-engine           | Limitado (Spark)  | ✅ (Spark, Flink, Trino, Hive) |
| Catálogo               | Spark Catalog     | Múltiplos (Hive, REST, Glue, Nessie) |
| Criado por             | Databricks        | Netflix / Apache  |
| Licença                | Apache 2.0        | Apache 2.0        |

---

## Versões de Compatibilidade

| Iceberg Runtime                         | Spark | Java |
|----------------------------------------|-------|------|
| iceberg-spark-runtime-3.5_2.12:1.4.3  | 3.5.x | 8+   |
| iceberg-spark-runtime-3.4_2.12:1.4.3  | 3.4.x | 8+   |
| iceberg-spark-runtime-3.3_2.12:1.4.3  | 3.3.x | 8+   |
