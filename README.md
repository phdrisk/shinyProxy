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
