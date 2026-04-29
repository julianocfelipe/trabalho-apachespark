# Apache Spark com Delta Lake e Apache Iceberg

Trabalho de Pesquisa da disciplina de **Engenharia de Dados** — SATC 2026.

Demonstra operações ACID (INSERT, UPDATE, DELETE) com **Delta Lake** e **Apache Iceberg** usando **PySpark**, em um cenário de Loja Virtual.

> **Documentação completa:** [julianocfelipe.github.io/trabalho-apachespark](https://julianocfelipe.github.io/trabalho-apachespark)

---

## Pré-requisitos

| Requisito | Versão        | Download |
|-----------|--------------|----------|
| Python    | 3.11         | [python.org](https://www.python.org/downloads/) |
| Java (JDK)| 8 ou 11      | [adoptium.net](https://adoptium.net/) |
| uv        | latest       | `pip install uv` |
| Hadoop (Windows) | winutils | [github.com/steveloughran/winutils](https://github.com/steveloughran/winutils) |

> **Windows:** É necessário configurar o Hadoop para o Spark funcionar. Instale o `winutils.exe` em `C:\hadoop\bin\`.

---

## Instalação

### 1. Clone o repositório

```bash
git clone https://github.com/julianocfelipe/trabalho-apachespark.git
cd trabalho-apachespark
```

### 2. Crie e ative o ambiente virtual com uv

```bash
# Cria o ambiente virtual e instala as dependências
uv sync
```

> O `uv sync` lê o `pyproject.toml` e instala todas as dependências no `.venv/`.

### 3. Configure a variável HADOOP_HOME (Windows)

Faça o download do `winutils.exe` compatível com Hadoop 3.x e coloque em `C:\hadoop\bin\`.

```bash
# Crie a pasta
mkdir C:\hadoop\bin

# Baixe o winutils.exe (Hadoop 3.3.5) e mova para C:\hadoop\bin\
```

Adicione ao PATH do sistema:
```
HADOOP_HOME = C:\hadoop
PATH += C:\hadoop\bin
```

> Já está configurado via código nos notebooks: `os.environ["HADOOP_HOME"] = "C:\\hadoop"`

---

## Como Executar

### Opção 1: JupyterLab (recomendado)

```bash
# Inicia o JupyterLab usando o ambiente virtual
uv run jupyter lab
```

Abra os notebooks na interface do JupyterLab:
- `notebooks/delta_lake.ipynb` — Demonstração do Delta Lake
- `notebooks/iceberg.ipynb` — Demonstração do Apache Iceberg

Execute todas as células em ordem (Kernel > Restart & Run All).

### Opção 2: VS Code com extensão Jupyter

1. Abra a pasta `trabalho-apachespark` no VS Code
2. Selecione o kernel Python do `.venv`
3. Abra e execute os notebooks

---

## Estrutura do Projeto

```
trabalho-apachespark/
├── README.md               # Este arquivo
├── pyproject.toml          # Dependências gerenciadas pelo uv
├── uv.lock                 # Lockfile de dependências
├── .python-version         # Versão do Python (3.11)
├── .gitignore              # Arquivos ignorados pelo Git
├── mkdocs.yml              # Configuração da documentação MKDocs
│
├── notebooks/
│   ├── delta_lake.ipynb    # Notebook: Delta Lake com PySpark
│   └── iceberg.ipynb       # Notebook: Apache Iceberg com PySpark
│
└── docs/
    ├── index.md            # Contextualização do trabalho
    ├── spark.md            # Explicação do Apache Spark / PySpark
    ├── delta.md            # Explicação do Delta Lake
    └── iceberg.md          # Explicação do Apache Iceberg
```

---

## Dependências

```toml
[project]
dependencies = [
    "delta-spark==3.0.0",    # Delta Lake para Spark 3.5
    "pyspark==3.5.0",        # Apache Spark
    "ipykernel>=7.2.0",      # Kernel Jupyter
    "jupyterlab>=4.5.6",     # Interface JupyterLab
    "mkdocs>=1.6.0",         # Gerador de documentação
    "mkdocs-material>=9.5.0" # Tema Material para MKDocs
]
```

> O Apache Iceberg é carregado automaticamente via `spark.jars.packages` no notebook, sem necessidade de instalação separada.

---

## Documentação MKDocs

### Visualizar localmente

```bash
uv run mkdocs serve
```

Acesse: [http://127.0.0.1:8000](http://127.0.0.1:8000)

### Publicar no GitHub Pages

```bash
uv run mkdocs gh-deploy
```

A documentação será publicada em: `https://julianocfelipe.github.io/trabalho-apachespark`

---

## Conteúdo dos Notebooks

Ambos os notebooks demonstram as mesmas operações no mesmo cenário (Loja Virtual):

| Seção | Operação | Descrição |
|-------|----------|-----------|
| 1 | Configuração | SparkSession com Delta/Iceberg |
| 2 | DDL | `CREATE TABLE` com schema explícito |
| 3 | INSERT | Inserção de clientes, produtos e pedidos |
| 4 | UPDATE | Reajuste de preços e atualização de status |
| 5 | DELETE | Remoção de produtos sem estoque |
| 6 | Analytics | Consultas analíticas sobre os dados |

---

## Tecnologias

![Apache Spark](https://img.shields.io/badge/Apache%20Spark-3.5.0-E25A1C?logo=apachespark&logoColor=white)
![Delta Lake](https://img.shields.io/badge/Delta%20Lake-3.0.0-00ADD8)
![Apache Iceberg](https://img.shields.io/badge/Apache%20Iceberg-1.4.3-4CAF50)
![Python](https://img.shields.io/badge/Python-3.11-3776AB?logo=python&logoColor=white)
![uv](https://img.shields.io/badge/uv-package%20manager-DE5FE9)

---

## Referências

### Apache Spark / PySpark
- Apache Spark — documentação oficial: https://spark.apache.org/docs/3.5.0/
- PySpark API Reference: https://spark.apache.org/docs/3.5.0/api/python/
- Zaharia, M. et al. *Apache Spark: A Unified Engine for Big Data Processing*. Communications of the ACM, 2016. https://doi.org/10.1145/2934664

### Delta Lake
- Delta Lake — documentação oficial: https://docs.delta.io/3.0.0/index.html
- Delta Lake no GitHub: https://github.com/delta-io/delta
- Armbrust, M. et al. *Delta Lake: High-Performance ACID Table Storage over Cloud Object Stores*. VLDB, 2020. https://doi.org/10.14778/3415478.3415560

### Apache Iceberg
- Apache Iceberg — documentação oficial: https://iceberg.apache.org/docs/1.4.3/
- Apache Iceberg — especificação do formato: https://iceberg.apache.org/spec/
- Apache Iceberg no GitHub: https://github.com/apache/iceberg
- Ryan Blue et al. *Iceberg: A Fast Table Format for S3*. SIGMOD, 2017. https://doi.org/10.1145/3183713.3190664

### Ferramentas e Ambiente
- uv — gerenciador de pacotes Python: https://docs.astral.sh/uv/
- JupyterLab — documentação: https://jupyterlab.readthedocs.io/en/stable/
- MkDocs — gerador de documentação: https://www.mkdocs.org/
- MkDocs Material Theme: https://squidfunk.github.io/mkdocs-material/
- Winutils para Hadoop no Windows: https://github.com/steveloughran/winutils

### Assistência
- Claude (Anthropic) — auxílio em pesquisas e estruturação do projeto: https://claude.ai
