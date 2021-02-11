# shinyProxy
## comandos necessarios
```
systemctl daemon-reload
systemctl restart docker
```
```
https://lukesingham.com/shiny-containers-with-shinyproxy/
```

## Arquivo application
```
/etc/shinyproxy/application.yml

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
    libssh2-1-dev

# system library dependency for the euler app
RUN apt-get update && apt-get install -y \
    libmpfr-dev

# basic shiny functionality
RUN R -e "install.packages(c('shiny', 'shinyMobile', 'shinyWidgets','shinycssloaders'), repos='https://cloud.r-project.org/')"

# basic  functionality
RUN R -e "install.packages(c('quantmod', 'quadprog', 'ggplot2','xts','dygraphs','utf8','stringr','BBmisc'), repos='https://cloud.r-project.org/')"

# basic  functionality
RUN R -e "install.packages(c('zeallot', 'DT', 'mFilters','tseries','lubridate','nnet','BBmisc'), repos='https://cloud.r-project.org/')"

# basic  functionality
## RUN R -e "install.packages(c('BETS','perfomanceAnalytics'), repos='https://cloud.r-project.org/')"

# copy the app to the image
RUN mkdir /root/webapp
COPY webapp /root/webapp

# configucao da porta
COPY Rprofile.site /usr/lib/R/etc/

EXPOSE 3838

CMD ["R", "-e", "shiny::runApp('/root/webapp')"]
```
