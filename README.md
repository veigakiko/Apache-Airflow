# Apache Airflow Docker Setup Guide

Este guia fornecerá um passo a passo extremamente detalhado para configurar, criar e desenvolver o serviço do Apache Airflow usando Docker no Windows 10. Todas as etapas serão minuciosamente descritas, incluindo scripts, boas práticas e códigos necessários para execução. Ao final, pergunte se todas as etapas foram aprovadas.

## Pré-requisitos

Antes de começarmos, certifique-se de ter os seguintes pré-requisitos:

- **Sistema Operacional:** Windows 10
- **Docker Desktop:** Certifique-se de ter o Docker Desktop instalado e configurado, habilitando o WSL 2 como backend.
- **WSL 2:** Subsystem para Linux no Windows, com a distribuição do Ubuntu instalada.
- **Git:** Ter o Git instalado para clonar o repositório e fazer commits.
- **Visual Studio Code (VSCode):** Recomendado para editar arquivos do projeto e instalar extensões como Docker e Python para facilitar o desenvolvimento.
- **Python 3.8+**: Para realizar testes e criar scripts adicionais fora do contêiner do Airflow.

## Etapa 1: Clonar o Repositório do GitHub

Comece clonando o repositório `Apache-Airflow` do GitHub para o seu ambiente local:

```sh
# No terminal (Git Bash ou Ubuntu no WSL)
git clone https://github.com/SEU_USUARIO/Apache-Airflow.git
cd Apache-Airflow
```

## Etapa 2: Estrutura do Projeto

Crie uma estrutura de pastas para organizar os arquivos do projeto. No diretório `Apache-Airflow`, crie as pastas:

```sh
mkdir dags logs plugins config
```

- **dags/**: Contém seus DAGs (gráficos acíclicos direcionados).
- **logs/**: Contém os arquivos de log do Airflow.
- **plugins/**: Contém plugins personalizados para o Airflow.
- **config/**: Contém arquivos de configuração adicionais, como `airflow.cfg` personalizado.

## Etapa 3: Crie o Arquivo docker-compose.yml

No diretório raiz do projeto, crie um arquivo chamado `docker-compose.yml`. Este arquivo define os serviços necessários para executar o Airflow:

```yaml
version: '3.8'

services:
  postgres:
    image: postgres:13
    container_name: postgres_airflow
    restart: always
    environment:
      POSTGRES_USER: airflow
      POSTGRES_PASSWORD: airflow
      POSTGRES_DB: airflow
    volumes:
      - postgres_data:/var/lib/postgresql/data
    networks:
      - airflow_net

  airflow:
    image: apache/airflow:2.5.1
    container_name: airflow_container
    restart: always
    environment:
      AIRFLOW__CORE__EXECUTOR: LocalExecutor
      AIRFLOW__CORE__SQL_ALCHEMY_CONN: 'postgresql+psycopg2://airflow:airflow@postgres_airflow:5432/airflow'
      AIRFLOW__CORE__LOAD_EXAMPLES: 'false'
    volumes:
      - ./dags:/opt/airflow/dags
      - ./logs:/opt/airflow/logs
      - ./plugins:/opt/airflow/plugins
    ports:
      - 8080:8080
    depends_on:
      - postgres
    command: >
      bash -c "airflow db init && airflow users create \
        --username admin \
        --firstname Ricardo \
        --lastname Admin \
        --role Admin \
        --email admin@example.com \
        --password admin && airflow scheduler & airflow webserver"
    networks:
      - airflow_net

networks:
  airflow_net:
    driver: bridge

volumes:
  postgres_data:
    driver: local
```

Este arquivo `docker-compose.yml` define os contêineres do Airflow e do Postgres, especifica a imagem do Airflow, os volumes e as variáveis de ambiente. Certifique-se de substituir `admin@example.com` e `admin` com suas próprias credenciais.

## Etapa 4: Inicializar o Docker Compose

Agora é hora de inicializar o Airflow com Docker Compose. No terminal, execute:

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
docker cp airflow_container:/opt/airflow/airflow.cfg ./config/airflow.cfg
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
docker exec -it airflow_container airflow connections add 'my_postgres' --conn-uri 'postgresql+psycopg2://user:password@host:5432/dbname'
```

## Etapa 9: Configurar Variáveis do Airflow

Configurar variáveis é importante para armazenar informações sensíveis e que podem mudar de ambiente para ambiente. Para configurar variáveis, você pode:

1. Navegar até a interface web do Airflow.
2. Ir para "Admin" > "Variables".
3. Adicionar uma nova variável conforme necessário.

Alternativamente, você pode usar a linha de comando dentro do contêiner:

```sh
docker exec -it airflow_container airflow variables set MY_VARIABLE my_value
```

## Etapa 10: Criar Scripts de Backup e Restauração

Para garantir a segurança dos dados do Airflow, crie scripts para backup e restauração dos bancos de dados e arquivos essenciais:

- **Backup do Banco de Dados PostgreSQL**:

```sh
docker exec -t postgres_airflow pg_dumpall -c -U airflow > ./backups/airflow_backup.sql
```

- **Restauração do Banco de Dados PostgreSQL**:

```sh
docker exec -i postgres_airflow psql -U airflow -d airflow < ./backups/airflow_backup.sql
```

## Etapa 11: Monitorar e Otimizar o Desempenho

- **Monitoramento dos Logs**: Verifique regularmente os logs do Airflow para identificar possíveis erros:

```sh
docker-compose logs -f airflow
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

- **Erro "Port already in use"**: Verifique se não há outro processo usando a porta 8080.
- **Problemas de Permissão**: Execute o Docker Desktop como administrador.
- **Recursos do Sistema**: Certifique-se de que sua configuração de Docker Desktop aloca memória suficiente para o contêiner.
- **Falha ao Iniciar o Scheduler**: Verifique as dependências e certifique-se de que o banco de dados está acessível.

## Etapa 13: Subir o Repositório para o GitHub

Depois de garantir que tudo está funcionando, commit e envie as mudanças para o seu repositório no GitHub:

```sh
git add .
git commit -m "Configuração do Apache Airflow com Docker"
git push origin main
```

## Etapa Final: Aprovado?

Estas foram todas as etapas para configurar e desenvolver o Apache Airflow usando Docker no Windows 10, seguindo boas práticas de engenharia de dados. Agora gostaria de saber se o passo a passo está aprovado ou se precisa de algum ajuste.

## Licença
Este projeto está licenciado sob a licença MIT - veja o arquivo LICENSE para mais detalhes.
