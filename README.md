# 🚀 Projeto Airflow + dbt - Jornada Data Warehouse

Este projeto implementa um pipeline de dados completo utilizando **Apache Airflow** para orquestração e **dbt** para transformações, criando um data warehouse moderno para análise de vendas e métricas de clientes.

## Visão geral

O projeto simula um e-commerce com dados de clientes e pedidos, processando informações através de um pipeline ELT (Extract, Load, Transform) que inclui:

- **Extração**: Dados de clientes e pedidos em formato CSV
- **Carga**: Armazenamento em PostgreSQL
- **Transformação**: Modelos dbt organizados em camadas (staging → intermediate → mart)
- **Orquestração**: DAGs do Airflow para execução automatizada

## Arquitetura

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Dados CSV     │───▶│   PostgreSQL    │───▶│   dbt Models    │
│   (Seeds)       │    │   (Raw Data)    │    │   (Transform)   │
└─────────────────┘    └─────────────────┘    └─────────────────┘
                                                       │
                       ┌─────────────────┐            │
                       │   Airflow DAG   │◀───────────┘
                       │  (Orquestração) │
                       └─────────────────┘
```

## Estrutura do projeto

```
airflow_dbt_project/
├── airflow/                          # Configuração do Airflow
│   ├── dags/
│   │   └── dag.py                   # DAG principal com dbt
│   ├── dbt/jornada_dw/              # Projeto dbt
│   │   ├── models/
│   │   │   ├── staging/             # Camada de staging
│   │   │   │   ├── stg_cadastros.sql
│   │   │   │   └── stg_pedidos.sql
│   │   │   ├── intermediate/        # Camada intermediária
│   │   │   │   ├── dim/
│   │   │   │   │   ├── int_dim_clientes.sql
│   │   │   │   │   └── int_dim_date.sql
│   │   │   │   └── fact/
│   │   │   │       └── int_fact_pedidos.sql
│   │   │   └── mart/                # Camada de mart
│   │   │       ├── mart_vendas_por_periodo.sql
│   │   │       └── mart_metricas_clientes.sql
│   │   ├── seeds/                   # Dados de exemplo
│   │   │   ├── cadastros.csv
│   │   │   └── pedidos.csv
│   │   └── dbt_project.yml
│   └── requirements.txt
├── local_setup/                     # Setup local com Docker
│   ├── docker-compose.yml
│   ├── pyproject.toml
│   └── generate_fake_data.py
└── data_warehouse/                  # Cópia do projeto dbt
```

## Funcionalidades

### Modelos de dados

#### Camada staging
- **`stg_cadastros`**: Dados limpos de clientes
- **`stg_pedidos`**: Dados limpos de pedidos com cálculos de valor total

#### Camada intermediate
- **`int_dim_clientes`**: Dimensão de clientes com SCD Type 1
- **`int_dim_date`**: Dimensão de datas para análises temporais
- **`int_fact_pedidos`**: Tabela de fatos principal

#### Camada mart
- **`mart_vendas_por_periodo`**: Métricas de vendas com análises temporais
- **`mart_metricas_clientes`**: Métricas específicas de clientes

### 🔄 Pipeline de dados

1. **Extração**: Dados CSV são carregados como seeds no dbt
2. **Staging**: Limpeza e padronização dos dados brutos
3. **Intermediate**: Criação de dimensões e fatos
4. **Mart**: Agregações e métricas de negócio
5. **Orquestração**: Execução diária via Airflow

## Como Executar

### Pré-requisitos

- Docker e Docker Compose
- Python 3.11+ (para setup local)
- Poetry (opcional, para gerenciamento de dependências)

### 1. Setup Llcal com Docker

```bash
# Clone o repositório
git clone https://github.com/gabriel-garciae/airflow_dbt_project
cd airflow_dbt_project

# Navegue para o diretório local_setup
cd local_setup

# Configure as variáveis de ambiente
export DBT_USER=postgres
export DBT_PASSWORD=postgres

# Inicie o PostgreSQL
docker-compose up -d

# Instale as dependências Python (opcional)
poetry install
```

### 2. Executar dbtlLocalmente

```bash
# Navegue para o diretório do projeto dbt
cd ../airflow/dbt/jornada_dw

# Instale as dependências do dbt
dbt deps

# Execute os seeds (dados de exemplo)
dbt seed

# Execute os modelos
dbt run

# Execute os testes
dbt test
```

### 3. Executar com Airflow

```bash
# Navegue para o diretório do Airflow
cd ../airflow

# Instale as dependências
pip install -r requirements.txt

# Configure as variáveis de ambiente do Airflow
export AIRFLOW_HOME=$(pwd)
export DBT_USER=postgres
export DBT_PASSWORD=postgres

# Inicialize o banco de dados do Airflow
airflow db init

# Crie um usuário admin
airflow users create \
    --username admin \
    --firstname Admin \
    --lastname User \
    --role Admin \
    --email admin@example.com

# Inicie o webserver
airflow webserver --port 8080

# Em outro terminal, inicie o scheduler
airflow scheduler
```

## Dados de exemplo

O projeto inclui dados sintéticos gerados com Faker:

- **10.000+ clientes** com informações demográficas e de contato
- **50.000+ pedidos** com valores, endereços e status
- **Período**: Dados de 2024-2025 para análises temporais

## Configurações

### Variáveis de Ambiente

```bash
# Banco de dados
DBT_USER=postgres
DBT_PASSWORD=postgres

# Airflow
AIRFLOW_HOME=/caminho/para/airflow
dbt_env=dev  # ou 'prod' para produção
```

### Perfis dbt

O projeto está configurado para dois ambientes:
- **dev**: PostgreSQL local via Docker
- **prod**: PostgreSQL na Railway (configuração de exemplo)

## Métricas disponíveis

### Vendas por período
- Receita bruta diária/mensal
- Ticket médio
- Número de pedidos
- Clientes únicos
- Crescimento percentual
- Médias móveis (7 dias)
- Análises sazonais

### Métricas de clientes
- Segmentação por valor
- Análise de recência
- Padrões de compra

## Testes

O projeto inclui testes dbt para garantir a qualidade dos dados:

```bash
# Executar todos os testes
dbt test

# Executar testes específicos
dbt test --select test_type:singular
dbt test --select test_type:generic
```

## Monitoramento

- **Airflow UI**: http://localhost:8080
- **Logs**: Disponíveis em `airflow/logs/`
- **dbt Docs**: Gere com `dbt docs generate && dbt docs serve`

## Tecnologias utilizadas

- **Apache Airflow**: Orquestração de workflows
- **dbt**: Transformações de dados
- **PostgreSQL**: Banco de dados
- **Docker**: Containerização
- **Python**: Linguagem principal
- **Astronomer Cosmos**: Integração Airflow + dbt

## Próximos Passos

- [ ] Implementar testes de qualidade de dados mais robustos
- [ ] Adicionar alertas e notificações
- [ ] Configurar CI/CD
- [ ] Implementar data quality checks
- [ ] Adicionar mais métricas de negócio
- [ ] Configurar backup e recovery

## Contribuição

1. Fork o projeto
2. Crie uma branch para sua feature (`git checkout -b feature/nova-feature`)
3. Commit suas mudanças (`git commit -am 'Adiciona nova feature'`)
4. Push para a branch (`git push origin feature/nova-feature`)
5. Abra um Pull Request


## Autores

- **Gabriel Evangelista** - [eng.gabrielgevangelista@gmail.com](mailto:eng.gabrielgevangelista@gmail.com)

---

**Nota**: Este é um projeto de demonstração criado para fins educacionais e de aprendizado em engenharia de dados.