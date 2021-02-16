# shinyProxy
## comandos necessarios
```
docker build -t 'nomeNovaImagem' .
systemctl daemon-reload
systemctl restart docker

variavel de ambiente
SHINYPROXY_USERNAME: o nome do usuário, conforme usado ao fazer login
SHINYPROXY_USERGROUPS: os grupos dos quais o usuário autenticado é membro, como um valor separado por vírgula
SHINYPROXY_OIDC_ACCESS_TOKEN: o token de acesso de conexão OpenId que pode ser reutilizado para invocar outra API de dentro do aplicativo Shiny

USO: 
userName <- Sys.getenv("SHINYPROXY_USERNAME")
```
```
https://lukesingham.com/shiny-containers-with-shinyproxy/
```

## Arquivo application
```
/etc/shinyproxy/application.yml

proxy:
  title: Open Analytics Shiny Proxy
  logo-url: https://www.openanalytics.eu/shinyproxy/logo.png
  landing-page: /
  heartbeat-rate: 10000
  heartbeat-timeout: 60000
  port: 8080
  authentication: ldap
  admin-groups: scientists
  # Example: 'simple' authentication configuration
  users:
  - name: jack
    password: password
    groups: scientists
  - name: jeff
    password: password
    groups: mathematicians
  # Example: 'ldap' authentication configuration
  ldap:
    url: ldap://ldap.forumsys.com:389/dc=example,dc=com
    user-dn-pattern: uid={0}
    group-search-base:
    group-search-filter: (uniqueMember={0})
    manager-dn: cn=read-only-admin,dc=example,dc=com
    manager-password: password
  # Docker configuration
  docker:
    cert-path: /home/none
    url: http://localhost:2375
    port-range-start: 20000
  specs:
  - id: 01_hello
    display-name: Hello Application
    description: Application which demonstrates the basics of a Shiny app
    container-cmd: ["R", "-e", "shinyproxy::run_01_hello()"]
    container-image: openanalytics/shinyproxy-demo
    access-groups: [scientists, mathematicians]
  - id: 06_tabsets
    container-cmd: ["R", "-e", "shinyproxy::run_06_tabsets()"]
    container-image: openanalytics/shinyproxy-demo
    
    MAPEANDO O VOLUME
    container-volumes: [nomeVolume:/pastaDentrodo App] ( criado no dockerFile *1)
    ### Ex. container-volumes[mercadoFinanceiro:/root/webapp/dados]
    
    access-groups: scientists

logging:
  file:
    name: shinyproxy.log



```
## ERRO JAVA 
```
edit /lib/systemd/system/docker.service
ExecStart=/usr/bin/dockerd -H unix:// -D -H tcp://127.0.0.1:2375
````
# DOCKERFILE
```
FROM openanalytics/r-base

LABEL maintainer "LUIZ CARLOS MARTINS <phdrisk@phdrisk.com.br>"

# system libraries of general use
RUN apt-get update && apt-get install -y \
    sudo \
    pandoc \
    pandoc-citeproc \
    libcurl4-gnutls-dev \
    libcairo2-dev \
    libxt-dev \
    libssl-dev \
    libssh2-1-dev \
    libxml2  \ # devtools
    libxml2-dev
    libmysql++-dev #BETS
# system library dependency for the euler app
RUN apt-get update && apt-get install -y \
    libmpfr-dev

# basic shiny functionality
RUN R -e "install.packages(c('shiny', 'shinyMobile', 'shinyWidgets','shinycssloaders'), repos='https://cloud.r-project.org/')"



# basic  functionality
# NECESSARIO PARA O QUANTMOD
RUN sudo apt-get install -y curl libcurl4-openssl-dev libssl-dev
RUN R -e "install.packages(c('quantmod', 'quadprog', 'ggplot2','xts','dygraphs','utf8','stringr','BBmisc'), repos='https://cloud.r-project.org/')"

# basic  functionality
RUN R -e "install.packages(c('zeallot', 'DT', 'mFilter','tseries','lubridate','nnet'), repos='https://cloud.r-project.org/')"

# basic  functionality
## RUN R -e "install.packages(c('xml2','BETS','PerfomanceAnalytics'), repos='https://cloud.r-project.org/')"

# copy the app to the image
RUN mkdir /root/webapp
COPY webapp /root/webapp

RUN mkdir /root/webapp/dados (*1) -> criado para o volume


# configucao da porta
COPY Rprofile.site /usr/lib/R/etc/

EXPOSE 3838

CMD ["R", "-e", "shiny::runApp('/root/webapp')"]
```
