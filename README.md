
# Apache Airflow - Docker Setup no Windows 10

![GitHub repo size](https://img.shields.io/github/repo-size/seu-usuario/Apache-Airflow?style=for-the-badge)
![GitHub language count](https://img.shields.io/github/languages/count/seu-usuario/Apache-Airflow?style=for-the-badge)
![GitHub forks](https://img.shields.io/github/forks/seu-usuario/Apache-Airflow?style=for-the-badge)
![Bitbucket open issues](https://img.shields.io/bitbucket/issues/seu-usuario/Apache-Airflow?style=for-the-badge)
![Bitbucket open pull requests](https://img.shields.io/bitbucket/pr-raw/seu-usuario/Apache-Airflow?style=for-the-badge)

<img src="airflow-setup.png" alt="Exemplo do Airflow Docker">

> Este projeto demonstra a configuração do Apache Airflow usando Docker no Windows 10, com integração ao WSL2 e suporte ao Docker Compose. O objetivo é fornecer um ambiente local de desenvolvimento eficiente e fácil de configurar.

### Ajustes e melhorias

O projeto ainda está em desenvolvimento e as próximas atualizações incluirão:

- [x] Adição do Docker Compose para Apache Airflow
- [x] Script de configuração para permissões
- [ ] Melhorias na documentação de problemas comuns
- [ ] Inclusão de DAGs de exemplo
- [ ] Configuração para ambientes de produção

## 💻 Pré-requisitos

Antes de começar, verifique se você atendeu aos seguintes requisitos:

- Você instalou o [Docker Desktop](https://www.docker.com/products/docker-desktop) com suporte ao WSL2.
- Você está rodando o Windows 10 com WSL2 habilitado.
- Você leu a documentação oficial do [Apache Airflow](https://airflow.apache.org/docs/apache-airflow/stable/start/docker.html).

## 🚀 Instalando Apache Airflow

Para instalar o Apache Airflow usando Docker no Windows 10, siga estas etapas:

### Linux e macOS:

```bash
git clone https://github.com/seu-usuario/Apache-Airflow.git
cd Apache-Airflow
bash airflow_settings.sh
docker-compose up -d
```

### Windows:

1. Clone o repositório:
   ```bash
   git clone https://github.com/seu-usuario/Apache-Airflow.git
   cd Apache-Airflow
   ```

2. Execute o script de configuração:
   ```bash
   bash airflow_settings.sh
   ```

3. Inicie o Docker Compose:
   ```bash
   docker-compose up -d
   ```

## ☕ Usando Apache Airflow

Para usar o Apache Airflow, siga estas etapas:

1. Acesse o Airflow Web UI via [http://localhost:8080](http://localhost:8080)
2. Login com as credenciais:
   - Usuário: `admin`
   - Senha: `admin`

3. Para verificar se os containers estão rodando corretamente:
   ```bash
   docker container ls
   ```

## 📫 Contribuindo para Apache Airflow

Para contribuir com o projeto, siga estas etapas:

1. Bifurque este repositório.
2. Crie um branch: `git checkout -b <nome_branch>`.
3. Faça suas alterações e confirme-as: `git commit -m '<mensagem_commit>'`
4. Envie para o branch original: `git push origin <nome_do_projeto>/<local>`
5. Crie uma solicitação de pull.

Como alternativa, consulte a documentação do GitHub em [como criar uma solicitação pull](https://help.github.com/en/github/collaborating-with-issues-and-pull-requests/creating-a-pull-request).

## 🤝 Colaboradores

Agradecemos às seguintes pessoas que contribuíram para este projeto:

<table>
  <tr>
    <td align="center">
      <a href="#" title="Ricardo Silva">
        <img src="https://avatars3.githubusercontent.com/u/31936044" width="100px;" alt="Foto do Ricardo Silva no GitHub"/><br>
        <sub>
          <b>Ricardo Silva</b>
        </sub>
      </a>
    </td>
    <td align="center">
      <a href="#" title="Mark Zuckerberg">
        <img src="https://s2.glbimg.com/FUcw2usZfSTL6yCCGj3L3v3SpJ8=/smart/e.glbimg.com/og/ed/f/original/2019/04/25/zuckerberg_podcast.jpg" width="100px;" alt="Foto do Mark Zuckerberg"/><br>
        <sub>
          <b>Mark Zuckerberg</b>
        </sub>
      </a>
    </td>
  </tr>
</table>

## 😄 Seja um dos contribuidores

Quer fazer parte desse projeto? Clique [AQUI](CONTRIBUTING.md) e leia como contribuir.

## 📝 Licença

Esse projeto está sob licença. Veja o arquivo [LICENÇA](LICENSE.md) para mais detalhes.
