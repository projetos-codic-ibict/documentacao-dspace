# Installing DSpace 7 from scratch

> [!IMPORTANT]
> This manual handles all directories location with `[]`, eg: `[dspace-install-dir]`. It means that you'll always to replace it wich real directory.
> 
> All values to be replaced inside `()` eg.: `(inform the user name)`, would be `EXAMPLE_USER`;



## Hardware pre requisites
- 8GB of RAM;
- Enough disk space consistent with the size that the repository will use.

## Software pre requisites
Install the following softwares on your S.O.

- Linux SO; 
- Java JDK 11;
- Maven 3.8 o higher (command `mvn`);
- Apache Ant 1.9.16 (command `ant`);
- Apache Tomcat 9;\
  The root directory of Apache Tomcat will be described as `[tomcat-dir]`;
- PostgreSQL version 12, with pgcrypto extension;
- Yarn (https://classic.yarnpkg.com/lang/en/docs/install/#debian-stable)
- Apache Solr 8.x 
  *Short guide to install apche solr:*
  - Download the Solr in: https://www.apache.org/dyn/closer.lua/lucene/solr/8.11.2/solr-8.11.2.zip?action=download
  - Unpack it in a directory, we will handle it as `[solr-dir]`;
  - Go to `[solr-dir]/bin` and execute the following command:
    
    ```
      ./solr start
    ```

## Intalling

### Backend

- Log-in in your PostgresSQL and create a new database:;
  
  ```sql
    CREATE DATABASE (nome da base de dados) OWNER (seu usuário do Postgres);
  ```
  
- Enable the extension `pgcrypto`:
  ```sql
  
  psql --username=postgres (nome da sua base) -c "CREATE EXTENSION pgcrypto;"

  ```
- Download the source code of DSpace 7.6: link: https://github.com/DSpace/DSpace/archive/refs/tags/dspace-7.6.zip
- Unpack the code in some directory, we will handle it as `[dspace-source-dir]`;
- Create a new direcotry as the dspace install dir, outside `[dspace-source-dir]`.

> [!NOTE]
> Usually the directory /dspcae is used to this purpose. Feel free to choose other directory.r outro local.

From now on, we will call this directory as `[dspace-install-dir]`.

**- Insert the variables into your configuration:**
  - Copy the file `[dspace-source-dir]/config/local.cfg.EXAMPLE` as destiny `[dspace-source-dir]/config/local.cfg` ;
  - Edit the file `[dspace-source-dir]/config/local.cfg` and register the appropriate to *at least* the foloowing variables bellow.
> [!NOTE]
> Those are minimum variables to get a DSpace running. You might need to add new variables. As SMTP server, and so on...

    ```
    dspace.dir=[dspace-install-dir]
    dspace.server.url=(inform the address where the back end will be avaiable)
    dspace.name=(inform the name of your repository)
    default.language=pt_BR
    
    dspace.ui.url=(iform the URL of the front-end: ex.: http://localhost:4000)
    ```

  **Database**
  Fullfill the variables above to allow DSpace to access the database
  
  ```
  db.url = jdbc:postgresql://(Database's IP):(Database's port)/(name of your Dspace database)
  db.username = (postgres username)
  db.password = (postgres password)
  ```

  **E-mail**

  If you have a SMTP server, to send e-mails, configure the variables in the section "Email configuration".

  **Other configurations:**
  Read other available configuration in file “local.cfg” and edit the ones that are appropiate for you.

**- Create de DSpace Install DIR**
  - Access the directory where you've unpacked the source code `cd [dspace-source-dir]`;
  - **Execute the command**
    ```
      mvn clean package
    ```
    *(wait for it, the first execution will take a long time)*

  - Access the directory `[dspace-source-dir]/dspace/target/dspace-installer`
  - Execute the command:
   ```
     ant fresh_install
   ```
- Create the first user:
  ```
  [dspace-install-dir]/bin/dspace create-administrator
  ```
- Create the Solr “cores”:
  ```
  [solr-dir]/bin/solr create -c search -d [dspace-install-dir]/solr/search/conf
  [solr-dir]/bin/solr create -c statistics -d [dspace-install-dir]/solr/statistics/conf
  [solr-dir]/bin/solr create -c oai -d [dspace-install-dir]/solr/oai/conf
  [solr-dir]/bin/solr create -c authority -d [dspace-install-dir]/solr/authority/conf
  ```

- Deploy the DSpace's backend in Tomcat
  - Do a symbolyc link of the generated wars in dspace dir to the Tomcat directory:
    ```
    ln -s  [dspace-install-dir]/webapps/server [tomcat-dir]/webapps/
    ln -s  [dspace-install-dir]/webapps/oai [tomcat-dir]/webapps/
    ```
- Start the Tomcat:
  ```
  [tomcat-dir]/bin/startup.sh
  ```
  All Tomcat logs will be avaiable in : `[tomcat-dir]/logs/catalina.out`

- Access the backend interface:
  Use the value registered for  `dspace.server.url` in file  `[dspace-source-dir]/dspace/local.cfg` plus suffix “/server”.
  Example: http://localhost:8080/server


### Frontend

- Download the source code in: https://github.com/DSpace/dspace-angular/archive/refs/tags/dspace-7.6.zip
- Unpack the front-end, from now on we will call it by  `[dspace-source-frontend-dir]` ;
- Copy the file  `[dspace-source-frontend-dir]/config/config.example.yaml` to  `[dspace-source-frontend-dir]/config/config.prod.yaml`  
- Edite the file `[dspace-source-frontend-dir]/config/config.prod.yaml` .
- Configure the variables as describle in the example bellow;
  ```
  ui:
    ssl: (inform tru if your site uses https)
    host: (set the address which the DSpace will be accessed by the user using the browser)
    port: (set the port where the front end will be avaiable)
    nameSpace: /
  ```

  ```
  rest:
    ssl: (inform tru if your back end uses https)
    host: (set o host defined in “local.cfg” of backend)
    port: (set o post defined in “local.cfg” of backend)
    nameSpace: /server
  ```

  (Optional) Bellow, in this file, fullfill the variable `defaultLanguage` with the appropriate locale:

  Exemplo:
  ```
  defaultLanguage: pt-BR
  ```

- Compile and run the project, executing the following commands:
  ```
  yarn install
  yearn start
  ```

  This proccess takes a while, when it finishes, access the front end using the URL registered above. Example: `http://lococalhost:8080`
  
