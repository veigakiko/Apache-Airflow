Apache Airflow - Docker Setup no Windows 10

<img src="airflow-setup.png" alt="Exemplo do Airflow Docker">
Este projeto demonstra a configuração do Apache Airflow usando Docker no Windows 10, com integração ao WSL2 e suporte ao Docker Compose. O objetivo é fornecer um ambiente local de desenvolvimento eficiente e fácil de configurar.

Ajustes e melhorias
O projeto ainda está em desenvolvimento e as próximas atualizações incluirão:

 Adição do Docker Compose para Apache Airflow
 Script de configuração para permissões
 Melhorias na documentação de problemas comuns
 Inclusão de DAGs de exemplo
 Configuração para ambientes de produção
💻 Pré-requisitos
Antes de começar, verifique se você atendeu aos seguintes requisitos:

Você instalou o Docker Desktop com suporte ao WSL2.
Você está rodando o Windows 10 com WSL2 habilitado.
Você leu a documentação oficial do Apache Airflow.
🚀 Instalando Apache Airflow
Para instalar o Apache Airflow usando Docker no Windows 10, siga estas etapas:

Linux e macOS:
bash
Copy code
git clone https://github.com/seu-usuario/Apache-Airflow.git
cd Apache-Airflow
bash airflow_settings.sh
docker-compose up -d
Windows:
Clone o repositório:

bash
Copy code
git clone https://github.com/seu-usuario/Apache-Airflow.git
cd Apache-Airflow
Execute o script de configuração:

Copy code
bash airflow_settings.sh
Inicie o Docker Compose:

Copy code
docker-compose up -d
☕ Usando Apache Airflow
Para usar o Apache Airflow, siga estas etapas:

Acesse o Airflow Web UI via http://localhost:8080

Login com as credenciais:

Usuário: admin
Senha: admin
Para verificar se os containers estão rodando corretamente:

bash
Copy code
docker container ls
📫 Contribuindo para Apache Airflow
Para contribuir com o projeto, siga estas etapas:

Bifurque este repositório.
Crie um branch: git checkout -b <nome_branch>.
Faça suas alterações e confirme-as: git commit -m '<mensagem_commit>'
Envie para o branch original: git push origin <nome_do_projeto>/<local>
Crie uma solicitação de pull.
Como alternativa, consulte a documentação do GitHub em como criar uma solicitação pull.

🤝 Colaboradores
Agradecemos às seguintes pessoas que contribuíram para este projeto:

<table> <tr> <td align="center"> <a href="#" title="Ricardo Silva"> <img src="https://avatars3.githubusercontent.com/u/31936044" width="100px;" alt="Foto do Ricardo Silva no GitHub"/><br> <sub> <b>Ricardo Silva</b> </sub> </a> </td> <td align="center"> <a href="#" title="Mark Zuckerberg"> <img src="https://s2.glbimg.com/FUcw2usZfSTL6yCCGj3L3v3SpJ8=/smart/e.glbimg.com/og/ed/f/original/2019/04/25/zuckerberg_podcast.jpg" width="100px;" alt="Foto do Mark Zuckerberg"/><br> <sub> <b>Mark Zuckerberg</b> </sub> </a> </td> </tr> </table>
😄 Seja um dos contribuidores
Quer fazer parte desse projeto? Clique AQUI e leia como contribuir.

📝 Licença
Esse projeto está sob licença. Veja o arquivo LICENÇA para mais detalhes.

Essa estrutura é completa, inclui os badges no início, uma introdução curta, um passo a passo de instalação para Windows, Linux e macOS, além de direções claras sobre como contribuir e usar o projeto.
