# Roteiro de instalação do DSpace 7.6

**Importante:**
- Este roteiro trata todas localizações de diretórios com “[]”, ex.: `[dspace-install-dir]`. Isso significa que você sempre terá que substituir essas marcações pelo real diretório;
- Todos valores a serem substituídos entre “()”, ex.: (informe o nome do usuário)


## Requisitos recomendados de hardware
- 8GB de memória RAM;
- Processador no mínimo dual core com pelo menos 1.5GHz;
- Espaço em disco condizente com o tamanho que o repositório fará uso;

## Pré requisitos de software
Instale os seguintes softwares no seu sistema operacional

- Sistema operacional Linux; 
- Java JDK 11;
- Maven 3.8 ou superior (comando `mvn`);
- Apache Ant 1.9.16 (comando `ant`);
- Apache Tomcat 9;\
  O diretório onde o Apache Tomcat está descompactado será chamado `[tomcat-dir]`
  Onde: `[tomcat-dir]` será tratado como diretório raiz do tomcat;
- Postgres versão 12, com a extensão pgcrypto habilitada;
- Yarn (https://classic.yarnpkg.com/lang/en/docs/install/#debian-stable)
- Apache Solr 8.x 
  *Orientações:*
  - Baixe o Solr em: https://www.apache.org/dyn/closer.lua/lucene/solr/8.11.2/solr-8.11.2.zip?action=download
  - Descompacte-o em local de sua preferência `[solr-dir]`;
  - Vá ao diretório `[solr-dir]/bin` e execute o seguinte comando:
    
    ```
      ./solr start
    ```

## Instalação

### Backend

- Crie uma base de dados no Postgres e habilite nela a extensão `pgcrypto`;
  
  ```
    CREATE DATABASE (nome da base de dados) OWNER (seu usuário do Postgres);
  ```
  
- Habilite a extensão pgcrypto nessa nova base:
  ```psql --username=postgres (nome da sua base) -c "CREATE EXTENSION pgcrypto;"```
- Baixe o código fonte do backend do DSpace 7.6, link: https://github.com/DSpace/DSpace/archive/refs/tags/dspace-7.6.zip
- Descompacte o código fonte do Backend em um diretório de preferência. 
- A partir de agora, chamaremos o diretório do código fonte descompactado de `[dspace-source-dir]`;
- Crie o diretório de instalação do DSpace, fora da estrutura do `[dspace-source-dir]`.
  
  Normalmente, é criado o diretório /dspace para este propósito. Fique à vontade para escolher outro local.

  A partir de agora chamaremos esse diretório de `[dspace-install-dir]`.

- Insira as variáveis do do seu repositório:
  - Copie o arquivo `[dspace-source-dir]/config/local.cfg.EXAMPLE` com destino ao arquivo  `[dspace-source-dir]/config/local.cfg` ;
  - Edite o arquivo `[dspace-source-dir]/config/local.cfg` e cadastre os valores adequados para as variáveis descritas abaixo.\
    Essas são as variáveis mínimas para que o DSpace funcione corretamente.

    ```
    dspace.dir=[dspace-install-dir]
    dspace.server.url=(informe o endereço onde o backend do seu dspace ficará disponível)
    dspace.name=(informe o nome do seu repositório)
    default.language=pt_BR
    
    dspace.ui.url=(informe a URL que o front end terá: ex.: http://localhost:4000)
    ```

  **Banco de dados**\
  Utilize o template abaixo para informar a URL de acesso ao postgres
  
  ```
  db.url = jdbc:postgresql://(IP do seu banco de dados):(porta de acesso ao banco de dados)/(nome da sua base de dados)
  db.
  db.username = (insira o nome do usuário do postgres)
  db.password = (insira a senha do usuário postgres)
  ```

  **E-mail**\

  Caso tenha um servidor SMTP para envio de e-mails, configure as variáveis na seção “EMAIL CONFIGURATION”.

  **Demais configurações:**\
  Leia as outras configurações disponíveis no arquivo “local.cfg” e edite as que achar conveniente.

- Preenchimento do diretório de instalação
  - Acesse o diretório onde descompactou o código fonte `[dspace-source-dir]`;
  - **Execute o comando**
    ```
      mvn clean package
    ```
    *(aguarde o processamento, a primeira execução demora)*

  - Acesse o diretório `[dspace-source-dir]/dspace/target/dspace-installer`
  - Digite o comando:
   ```
     ant fresh_install
   ```
- Crie o primeiro usuário, execute:
  ```
  [dspace-install-dir]/bin/dspace create-administrator
  ```
- Crie os “cores” do Solr, executando os comandos:
  ```
  [solr-dir]/bin/solr create -c search -d [dspace-install-dir]/solr/search/conf
  [solr-dir]/bin/solr create -c statistics -d [dspace-install-dir]/solr/statistics/conf
  [solr-dir]/bin/solr create -c oai -d [dspace-install-dir]/solr/oai/conf
  [solr-dir]/bin/solr create -c authority -d [dspace-install-dir]/solr/authority/conf
  ```

- Deploy do backend no Tomcat
  - Faça um link simbólico da aplicação gerada na aplicação gerada para o Tomcat:
    ```
    ln -s  [dspace-install-dir]/webapps/server [tomcat-dir]/webapps/
    ln -s  [dspace-install-dir]/webapps/oai [tomcat-dir]/webapps/
    ```
- Inicie o Tomcat:
  ```
  [tomcat-dir]/bin/startup.sh
  ```
  Aguarde alguns instantes para que o serviço inicie.\
  Os logs do Tomcat estão disponíveis em: `[tomcat-dir]/logs/catalina.out`

- Acesse a interface WEB do Backend:
  Utilize o endereço que foi preenchido na variavel `dspace.server.url` no arquivo  `[dspace-source-dir]/dspace/local.cfg` + sufixo “/server”.
  Exemplo: http://localhost:8080/server


### Frontend

- Baixe o código fonte do frontend do DSpace 7.6, link: https://github.com/DSpace/dspace-angular/archive/refs/tags/dspace-7.6.zip
- Descompacte o frontend, chamaremos o diretório raiz do front end de `[dspace-source-frontend-dir]` ;
- Copie o arquivo  `[dspace-source-frontend-dir]/config/config.example.yaml` para  `[dspace-source-frontend-dir]/config/config.prod.yaml`  
- Edite o arquivo `[dspace-source-frontend-dir]/config/config.prod.yaml` .
- Configure as variáveis conforme os escritos no exemplo abaixo:
  ```
  ui:
    ssl: (coloque true se seu site use https, false caso contrário)
    host: (coloque o endereço no qual a interface do seu DSpace será acessado pelo usuário)
    port: (indique a porta em que deseja disponibilizar a interface do DSpace)
    nameSpace: /
  ```

  ```
  rest:
    ssl: (coloque true caso o backend esteja em https)
    host: (coloque o host que foi definido no arquivo “local.cfg” do backend)
    port: (coloque a porta que foi definido no arquivo “local.cfg” do backend)
    nameSpace: /server
  ```

  Mais baixo no arquivo, preencha a variável `defaultLanguage` com “pt-BR”:
  ```
  defaultLanguage: pt_BR
  ```

- Compile e rode o projeto, executando os comandos:
  ```
  yarn install
  yearn start
  ```

  Este processo demora, ao final, acesse o frontend, usando o endereço cadastrado acima. Ex.: `http://lococalhost:8080`
  
