# Delta Lake

## O que é o Delta Lake?

O **Delta Lake** é uma camada de armazenamento open-source desenvolvida pela Databricks. Em resumo, ele pega arquivos Parquet comuns e adiciona suporte a transações **ACID**, controle de versão e schema enforcement por cima, sem precisar de um banco de dados relacional separado. Funciona em sistemas de arquivos como S3, ADLS, HDFS e até local.

---

## Principais Características

| Característica           | Descrição                                                                 |
|--------------------------|---------------------------------------------------------------------------|
| **ACID Transactions**    | Atomicidade, Consistência, Isolamento e Durabilidade nas operações        |
| **Scalable Metadata**    | Metadados armazenados em Spark, não em um servidor externo                |
| **Schema Enforcement**   | Rejeita dados que violam o schema definido                                |
| **Unified Batch/Stream** | Mesma API para processamento batch e streaming                            |
| **DML Completo**         | Suporte nativo a `UPDATE` e `DELETE`                                      |

---

## Arquitetura

O Delta Lake registra cada operação em um **Transaction Log** (Delta Log), que é basicamente uma pasta `_delta_log/` criada ao lado dos arquivos Parquet. Cada commit gera um novo arquivo JSON nessa pasta:

```
minha-tabela/
├── _delta_log/
│   ├── 00000000000000000000.json   ← versão 0 (CREATE/INSERT)
│   ├── 00000000000000000001.json   ← versão 1 (UPDATE)
│   ├── 00000000000000000002.json   ← versão 2 (DELETE)
│   └── ...
├── part-00000-abc.parquet
├── part-00001-def.parquet
└── ...
```

---

## Configuração com PySpark

### Instalação (uv)

```bash
uv add delta-spark==3.0.0 pyspark==3.5.0
```

### Inicializando a SparkSession com Delta

```python
from pyspark.sql import SparkSession
from delta import configure_spark_with_delta_pip

builder = SparkSession.builder \
    .appName("Delta Lake Demo") \
    .config("spark.sql.extensions",
            "io.delta.sql.DeltaSparkSessionExtension") \
    .config("spark.sql.catalog.spark_catalog",
            "org.apache.spark.sql.delta.catalog.DeltaCatalog")

spark = configure_spark_with_delta_pip(builder).getOrCreate()
```

---

## Operações DML

### DDL — Criar Tabela

```sql
CREATE TABLE IF NOT EXISTS produtos (
    id        INT,
    nome      STRING,
    categoria STRING,
    preco     DOUBLE,
    estoque   INT
) USING DELTA;
```

### INSERT — Inserir Dados

```python
spark.sql("""
    INSERT INTO produtos VALUES
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
# Reajuste de 10% nos preços de Eletrônicos
spark.sql("""
    UPDATE produtos
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

O DELETE remove fisicamente as linhas que atendem a condição. No exemplo abaixo, removemos produtos que estão sem estoque:

```python
spark.sql("""
    DELETE FROM produtos
    WHERE estoque = 0
""")
```

Após o DELETE, a tabela retém apenas os registros com `estoque > 0`. O Delta Log registra essa operação como uma nova versão da tabela.

---

## Versões de Compatibilidade

| Delta Lake | PySpark | Python  |
|-----------|---------|---------|
| 3.0.0     | 3.5.0   | 3.10+   |
| 2.4.0     | 3.4.0   | 3.10+   |
| 2.3.0     | 3.3.0   | 3.8+    |

---

## Delta Lake vs. Tabelas Parquet tradicionais

| Funcionalidade      | Parquet Puro | Delta Lake |
|--------------------|-------------|------------|
| UPDATE / DELETE    | ❌           | ✅          |
| ACID Transactions  | ❌           | ✅          |
| Schema Enforcement | ❌           | ✅          |
| Streaming          | Limitado     | ✅          |
