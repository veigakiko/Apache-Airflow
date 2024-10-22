Apache Airflow - Docker Setup no Windows 10
Disclaimer
As configurações descritas aqui são para fins de desenvolvimento local e estudos.

Pré-requisitos
Windows 10
Docker
WSL2 (Windows Subsystem for Linux)
Estrutura do Repositório
plaintext
Copy code
Apache-Airflow/
│
├── dags/                      # Diretório para DAGs
├── logs/                      # Diretório de logs do Airflow
├── plugins/                   # Diretório para plugins do Airflow
├── docker-compose.yml          # Arquivo Docker Compose
├── airflow_settings.sh         # Script de configuração
├── setup_instructions.md       # Instruções detalhadas
└── requirements.txt            # Dependências do Airflow
1. Configuração do Ambiente no Windows 10
1.1 Instalar o WSL2 (Windows Subsystem for Linux)
Para instalar o WSL2, siga as instruções oficiais da Microsoft: Guia de instalação do WSL2
1.2 Instalar Docker Desktop
Baixe e instale o Docker Desktop no Windows 10, certificando-se de habilitar o WSL2 no Docker. Download Docker Desktop
2. Criar e Configurar o Apache Airflow
2.1 Criar o arquivo docker-compose.yml
Este arquivo será usado para definir os serviços Docker para rodar o Airflow.

yaml
Copy code
version: '3.8'
services:
  airflow:
    image: apache/airflow:2.6.2
    container_name: airflow_container
    environment:
      - AIRFLOW__CORE__EXECUTOR=LocalExecutor
      - AIRFLOW__CORE__SQL_ALCHEMY_CONN=sqlite:////opt/airflow/airflow.db
      - AIRFLOW__CORE__LOAD_EXAMPLES=False
    volumes:
      - ./dags:/opt/airflow/dags
      - ./logs:/opt/airflow/logs
      - ./plugins:/opt/airflow/plugins
    ports:
      - "8080:8080"
    command: >
      bash -c "
      airflow db init &&
      airflow users create --username admin --password admin --firstname Ricardo --lastname Admin --role Admin --email admin@example.com &&
      airflow webserver & airflow scheduler"
    restart: always
2.2 Criar script de configuração airflow_settings.sh
Este script cria os diretórios e ajusta as permissões necessárias para que o Airflow funcione corretamente:

bash
Copy code
#!/bin/bash

# Criar diretórios para o Airflow
mkdir -p ./dags ./logs ./plugins

# Ajustar permissões
sudo chmod -R 777 ./dags ./logs ./plugins

echo "Permissões ajustadas e diretórios criados."
2.3 Criar o arquivo de dependências requirements.txt
plaintext
Copy code
apache-airflow==2.6.2
2.4 Instruções de Configuração e Execução
2.4.1 Clonar o repositório:
bash
Copy code
git clone https://github.com/seu-usuario/Apache-Airflow.git
cd Apache-Airflow
2.4.2 Executar o script de configuração:
bash
Copy code
bash airflow_settings.sh
2.4.3 Subir o serviço com Docker Compose:
bash
Copy code
docker-compose up -d
2.4.4 Acessar o Apache Airflow:
A interface web estará disponível em: http://localhost:8080
Login: admin
Senha: admin
3. Subir o Airflow e Verificar Logs
3.1 Verificando os containers criados
bash
Copy code
docker container ls
3.2 Verificar logs do Airflow:
bash
Copy code
docker logs airflow_container
4. Problemas Comuns e Soluções
O Docker não inicia:
Verifique se o WSL2 está habilitado nas configurações do Docker Desktop.
O Airflow não carrega as DAGs:
Certifique-se de que as DAGs estejam no diretório correto (./dags).
5. Conclusão
Com este repositório e as instruções detalhadas, você pode configurar rapidamente um ambiente de desenvolvimento do Apache Airflow no Windows 10 utilizando Docker e WSL2.


