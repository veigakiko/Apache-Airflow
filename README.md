# Apache Airflow Setup 

Este guia fornecerá um passo a passo extremamente detalhado para configurar, criar e desenvolver o serviço do Apache Airflow usando Docker no Windows 10. Todas as etapas serão minuciosamente descritas, incluindo scripts, boas práticas e códigos necessários para execução. Ao final, pergunte se todas as etapas foram aprovadas.

## Pré-requisitos

Antes de começarmos, certifique-se de ter os seguintes pré-requisitos:

- **Sistema Operacional:** Windows 10
- **Docker Desktop:** Certifique-se de ter o Docker Desktop instalado e configurado, habilitando o WSL 2 como backend.
- **WSL 2:** Subsystem para Linux no Windows, com a distribuição do Ubuntu instalada.
- **Git:** Ter o Git instalado para clonar o repositório e fazer commits.
- **Visual Studio Code (VSCode):** Recomendado para editar arquivos do projeto e instalar extensões como Docker e Python para facilitar o desenvolvimento.
- **Python 3.8+**: Para realizar testes e criar scripts adicionais fora do contêiner do Airflow.

## Etapa 1: Criar Ambiente Virtual e Clonar o Repositório do GitHub

1. **Criar o Ambiente Virtual**

   Vamos criar um ambiente virtual Python para garantir que todas as dependências estejam isoladas e que não haja conflitos com outras instalações Python em seu sistema.

   No diretório raiz do projeto, crie um ambiente virtual usando o comando:

   ```sh
   python -m venv airflow_venv
   ```

2. **Ativar o Ambiente Virtual**

   Ative o ambiente virtual:
   - **Windows**:
     ```sh
     .\airflow_venv\Scripts\activate
     ```
   - **Linux/WSL**:
     ```sh
     source airflow_venv/bin/activate
     ```

3. **Instalar Dependências**

   Atualize o `pip` e instale o `wheel`:

   ```sh
   pip install --upgrade pip
   pip install wheel
   ```

4. **Clonar o Repositório do GitHub**

   Agora clone o repositório `Apache-Airflow` do GitHub para o seu ambiente local:

   ```sh
   git clone https://github.com/SEU_USUARIO/Apache-Airflow.git
   cd Apache-Airflow
   ```

## Etapa 2: Estrutura do Projeto

Crie uma estrutura de pastas para organizar os arquivos do projeto. No diretório `Apache-Airflow`, crie as pastas:

```sh
mkdir dags logs plugins config postgres-data credentials
```

- **dags/**: Contém seus DAGs (gráficos acíclicos direcionados).
- **logs/**: Contém os arquivos de log do Airflow.
- **plugins/**: Contém plugins personalizados para o Airflow.
- **config/**: Contém arquivos de configuração adicionais, como `airflow.cfg` personalizado.
- **postgres-data/**: Armazena os dados do Postgres.
- **credentials/**: Contém as credenciais necessárias, como `bigquery_keyfile.json`.

## Etapa 2.1: Criar o Arquivo airflow.cfg Personalizado

O arquivo `airflow.cfg` contém configurações importantes para o funcionamento do Airflow. Vamos criar um arquivo personalizado para otimizar o desempenho e ajustar as configurações conforme suas necessidades específicas:

1. **Copiar o Arquivo Padrão de Configuração:** Primeiro, copie o arquivo padrão de configuração do Airflow de dentro do contêiner:

   ```sh
   docker cp airflow_webserver_1:/opt/airflow/airflow.cfg ./config/airflow.cfg
   ```

2. **Editar o Arquivo airflow.cfg:** Edite o arquivo `config/airflow.cfg` para ajustar os seguintes parâmetros:

   - **`parallelism`**: Define o número máximo de tarefas que podem ser executadas simultaneamente por toda a instância do Airflow. Ajuste este valor com base na capacidade de recursos disponíveis (ex.: `parallelism = 16`).
   
   - **`dag_concurrency`**: Número máximo de tarefas que podem ser executadas simultaneamente em um DAG específico. Defina um valor adequado para evitar sobrecarga (ex.: `dag_concurrency = 8`).

   - **`max_active_runs_per_dag`**: Controla quantas execuções de um DAG podem ser realizadas simultaneamente. Ajuste conforme necessário para evitar conflitos (ex.: `max_active_runs_per_dag = 2`).

   - **`load_examples`**: Definir como `False` para evitar o carregamento dos DAGs de exemplo que vêm por padrão com o Airflow, especialmente em ambientes de produção.

   - **`executor`**: Certifique-se de que está definido como `CeleryExecutor` para possibilitar o uso de vários workers e melhorar a escalabilidade (ex.: `executor = CeleryExecutor`).

   - **`sql_alchemy_pool_size`**: Define o tamanho da pool de conexões com o banco de dados. Aumente este valor se houver muitas conexões simultâneas (ex.: `sql_alchemy_pool_size = 10`).

   - **`worker_concurrency`**: Número de tarefas que cada worker pode processar simultaneamente. O valor ideal depende dos recursos de CPU/memória disponíveis (ex.: `worker_concurrency = 4`).

   - **`smtp_host`, `smtp_user`, `smtp_password`, `smtp_port`, `smtp_mail_from`**: Configure os parâmetros SMTP se desejar que o Airflow envie notificações por e-mail em caso de falha de tarefas ou outras situações.

   - **`default_timezone`**: Defina o fuso horário padrão do Airflow para garantir que os DAGs sejam executados em horários consistentes (ex.: `default_timezone = UTC`).

3. **Montar o Arquivo de Configuração no Contêiner:** Atualize o `docker-compose.yml` para montar este arquivo no contêiner do Airflow:

   ```yaml
       volumes:
         - ./config/airflow.cfg:/opt/airflow/airflow.cfg
   ```

4. **Boas Práticas ao Configurar airflow.cfg:**
   - Sempre mantenha um backup do arquivo original antes de fazer alterações significativas.
   - Ajuste os parâmetros de acordo com a capacidade de hardware disponível e as necessidades de escalabilidade do projeto.
   - Evite valores excessivamente altos para `parallelism` e `worker_concurrency`, pois podem sobrecarregar o banco de dados ou causar problemas de estabilidade.
   - Em ambientes de produção, configure as credenciais e parâmetros de segurança, como `fernet_key` (para criptografar variáveis de conexão sensíveis).

## Etapa 3: Crie o Arquivo docker-compose.yml

No diretório raiz do projeto, crie um arquivo chamado `docker-compose.yml`. Este arquivo define os serviços necessários para executar o Airflow:

```yaml
version: '3'

services:
  postgres:
    image: postgres:13
    environment:
      POSTGRES_USER: airflow
      POSTGRES_PASSWORD: airflow
      POSTGRES_DB: airflow
    volumes:
      - ./postgres-data:/var/lib/postgresql/data
    ports:
      - "5432:5432"  # Adicione esta linha para expor a porta 5432

  redis:
    image: redis:latest
    ports:
      - "6379:6379"

  webserver:
    image: apache/airflow:2.5.0
    environment:
      AIRFLOW__CORE__EXECUTOR: CeleryExecutor
      AIRFLOW__DATABASE__SQL_ALCHEMY_CONN: postgresql+psycopg2://airflow:airflow@postgres/airflow
      AIRFLOW__CELERY__BROKER_URL: redis://redis:6379/0
      AIRFLOW__CELERY__RESULT_BACKEND: db+postgresql://airflow:airflow@postgres/airflow
      GOOGLE_APPLICATION_CREDENTIALS: /opt/airflow/credentials/bigquery_keyfile.json
    ports:
      - "8080:8080"
    volumes:
      - ./dags:/opt/airflow/dags
      - ./logs:/opt/airflow/logs
      - ./plugins:/opt/airflow/plugins
      - ./credentials:/opt/airflow/credentials # Volume para as credenciais
    command: bash -c "pip install apache-airflow-providers-google requests beautifulsoup4 pandas && airflow db upgrade && exec airflow webserver"
    depends_on:
      - postgres
      - redis

  scheduler:
    image: apache/airflow:2.5.0
    environment:
      AIRFLOW__CORE__EXECUTOR: CeleryExecutor
      AIRFLOW__DATABASE__SQL_ALCHEMY_CONN: postgresql+psycopg2://airflow:airflow@postgres/airflow
      GOOGLE_APPLICATION_CREDENTIALS: /opt/airflow/credentials/bigquery_keyfile.json
    volumes:
      - ./dags:/opt/airflow/dags
      - ./logs:/opt/airflow/logs
      - ./plugins:/opt/airflow/plugins
      - ./credentials:/opt/airflow/credentials # Volume para as credenciais
    command: bash -c "pip install apache-airflow-providers-google requests beautifulsoup4 pandas && exec airflow scheduler"
    depends_on:
      - postgres
      - redis

  worker:
    image: apache/airflow:2.5.0
    environment:
      AIRFLOW__CORE__EXECUTOR: CeleryExecutor
      AIRFLOW__DATABASE__SQL_ALCHEMY_CONN: postgresql+psycopg2://airflow:airflow@postgres/airflow
      AIRFLOW__CELERY__BROKER_URL: redis://redis:6379/0
      AIRFLOW__CELERY__RESULT_BACKEND: db+postgresql://airflow:airflow@postgres/airflow
      GOOGLE_APPLICATION_CREDENTIALS: /opt/airflow/credentials/bigquery_keyfile.json
    volumes:
      - ./dags:/opt/airflow/dags
      - ./logs:/opt/airflow/logs
      - ./plugins:/opt/airflow/plugins
      - ./credentials:/opt/airflow/credentials # Volume para as credenciais
    command: bash -c "pip install apache-airflow-providers-google requests beautifulsoup4 pandas && exec airflow celery worker"
    depends_on:
      - postgres
      - redis

networks:
  default:
    driver: bridge
```

Este arquivo `docker-compose.yml` define os serviços do Airflow (webserver, scheduler e worker), além do banco de dados PostgreSQL e do broker Redis para o CeleryExecutor. Certifique-se de substituir `bigquery_keyfile.json` pelas suas credenciais do Google Cloud se necessário.

## Etapa 4: Inicializar o Docker Compose e Configurar o Ambiente

Agora é hora de inicializar o Airflow com Docker Compose. Antes de fazer isso, certifique-se de que o ambiente virtual criado esteja ativado para garantir que todas as dependências locais estejam disponíveis.

No terminal, execute:

```sh
docker-compose up -d
```

Isso iniciará os contêineres do Airflow e criará o banco de dados inicial.

- Para verificar os logs, use:

```sh
docker-compose logs -f
```

Certifique-se de que todos os contêineres foram iniciados sem erros. Caso encontre algum erro, consulte os logs e ajuste as configurações conforme necessário.

## Etapa 5: Acessar a Interface Web do Airflow

Depois que os contêineres estiverem em execução, você pode acessar a interface web do Airflow no navegador:

```
http://localhost:8080
```

Use o nome de usuário e senha especificados no `docker-compose.yml` (padrão: `admin` / `admin`).

## Etapa 6: Configurar o Airflow.cfg Personalizado

Altere as configurações do arquivo `airflow.cfg` para personalizar o comportamento do Airflow, como aumentar o limite de conexões ou ajustar a paralelização. Você pode copiar o arquivo padrão do contêiner:

```sh
docker cp airflow_webserver_1:/opt/airflow/airflow.cfg ./config/airflow.cfg
```

Edite o arquivo conforme as necessidades do seu projeto e monte este arquivo no contêiner:

```yaml
    volumes:
      - ./config/airflow.cfg:/opt/airflow/airflow.cfg
```

## Etapa 7: Criar um DAG Exemplo

Crie um arquivo de DAG de exemplo no diretório `dags` para testar o Airflow. Crie o arquivo `example_dag.py` com o seguinte conteúdo:

```python
from airflow import DAG
from airflow.operators.dummy import DummyOperator
from datetime import datetime

default_args = {
    'owner': 'airflow',
    'start_date': datetime(2023, 1, 1),
    'retries': 1
}

dag = DAG(
    'example_dag',
    default_args=default_args,
    description='A simple example DAG',
    schedule_interval='@daily',
    catchup=False
)

start = DummyOperator(
    task_id='start',
    dag=dag
)

end = DummyOperator(
    task_id='end',
    dag=dag
)

start >> end
```

Este exemplo define um DAG simples que é executado diariamente e possui duas tarefas dummy (`start` e `end`).

## Etapa 8: Configurar Conexões no Airflow

Para conectar o Airflow a outras ferramentas e fontes de dados, é necessário configurar conexões.

1. Navegue até a interface web do Airflow.
2. Vá para "Admin" > "Connections".
3. Clique em "Create" e configure a conexão conforme necessário (e.g., conexão com S3, banco de dados, API).

Alternativamente, você pode usar a linha de comando dentro do contêiner para configurar conexões:

```sh
docker exec -it airflow_webserver_1 airflow connections add 'my_postgres' --conn-uri 'postgresql+psycopg2://user:password@host:5432/dbname'
```

## Etapa 9: Configurar Variáveis do Airflow

Configurar variáveis é importante para armazenar informações sensíveis e que podem mudar de ambiente para ambiente. Para configurar variáveis, você pode:

1. Navegar até a interface web do Airflow.
2. Ir para "Admin" > "Variables".
3. Adicionar uma nova variável conforme necessário.

Alternativamente, você pode usar a linha de comando dentro do contêiner:

```sh
docker exec -it airflow_webserver_1 airflow variables set MY_VARIABLE my_value
```

## Etapa 10: Criar Scripts de Backup e Restauração

Para garantir a segurança dos dados do Airflow, crie scripts para backup e restauração dos bancos de dados e arquivos essenciais:

- **Backup do Banco de Dados PostgreSQL**:

```sh
docker exec -t postgres pg_dumpall -c -U airflow > ./backups/airflow_backup.sql
```

- **Restauração do Banco de Dados PostgreSQL**:

```sh
docker exec -i postgres psql -U airflow -d airflow < ./backups/airflow_backup.sql
```

## Etapa 11: Monitorar e Otimizar o Desempenho

- **Monitoramento dos Logs**: Verifique regularmente os logs do Airflow para identificar possíveis erros:

```sh
docker-compose logs -f webserver
```

- **Configurar Alerta por E-mail**: Configure uma conta de e-mail no arquivo `airflow.cfg` para que o Airflow envie alertas em caso de falhas.

## Etapa 12: Parar os Contêineres do Airflow

Depois de terminar o trabalho com o Airflow, você pode parar os contêineres executando:

```sh
docker-compose down
```

Se precisar remover os volumes criados, use:

```sh
docker-compose down -v
```

## Solução de Problemas Comuns

- **Erro "Port already in use"**: Verifique se não há
