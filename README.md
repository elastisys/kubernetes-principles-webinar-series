# kubernetes-principles-webinar-series

## Förberedelser

### Variablen `$DOMAIN`

Vi utgår i följande förberedelser från att det finns en variabel i ditt shell som heter `DOMAIN`, som brukligt är i Elastisys Kubernetes Platform.

### Harbor: skapa projekt och robot account för nginx-unprivileged

Logga in i din Harbor via `harbor.$DOMAIN`

Skapa ett nytt projekt, kalla det för `nginx-unprivileged`.

Gå sedan till dess Robot Accounts-del och skapa ett Robot Account som du ger namnet `kubernetes`.
Rutan bör se ut så här:

BILD

### Spegla container image för nginx-unprivileged

```sh
docker pull nginxinc/nginx-unprivileged:mainline-perl
docker tag nginxinc/nginx-unprivileged:mainline-perl harbor.$DOMAIN/nginx-unprivileged/nginx-unprivileged:mainline-perl
docker push harbor.$DOMAIN/nginx-unprivileged/nginx-unprivileged:mainline-perl
```


### Pull Secret

Skapa en "pull secret" för ditt Harbor-projekt genom att skriva kommandon nedan.
Vi antar att filen du laddade hem från Harbor heter `robot-account.json` (om det inte stämmer, döp om den så kommandona nedan fungerar).

```sh
DOCKER_USER=$(jq -r .name robot-account.json)
DOCKER_PASSWORD=$(jq -r .secret robot-account.json)

kubectl create secret docker-registry pull-secret-principles \
    --docker-server=harbor.$DOMAIN \
    --docker-username=$DOCKER_USER \
    --docker-password=$DOCKER_PASSWORD
```

