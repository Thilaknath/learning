## Creating and Running Containers in Azure

```
FROM alpine:latest

RUN mkdir /app
WORKDIR /app

COPY ./webapp/bin/release/publish ./
COPY ./config.sh ./

RUN bash config.sh

EXPOSE 80
ENTRYPOINT ["dotnet", "webapp.dl"] => Command followed by the executable to start
```

```
docker build --progress plain -t webappimage:v1 => progress command outputs whats happening inside container to terminal
```

### Azure Container Registry

Headless authentication is only possible through Azure Active Directory. Do not use Azure admin account for headless authentication. 

```
# Build using ACR Tasks
ACR_NAME='testACR'

az acr build --image "webimage:v1-acr-task" --registry $ACR_NAME .

# Create ACR
az group create \ 
    --name psdemo-rg \
    --location centralus

az acr create \
    --resource-group psdemo-rg \
    --name $ACR_NAME \
    --sku Standard


# Login to ACR 
az acr login --name $ACR_NAME

# Get list of repositories
az acr repository list --name $ACR_NAME --output table \
az acr repository show-tags --name $ACR_NAME --repository webappimage --output table
```

### Deploying to ACR
* Setup a Service Principal for ACI to Pull from ACR

```
ACR_NAME='testACR'
ACR_REGISTRY_ID=$(az acr show --name $ACR_NAME --query id --output tsv)

SP_NAME=acr-service-principal
SP_PASSWD=$(az ad sp create-for-rba \
    --name http://$ACR_NAME-pull \
    --scopes $ACR_REGISTRY_ID \
    --role acrpull \
    --query password \
    --output tsv)

SP_APPID=$(az ad sp show \
    --id http://$ACR_NAME-pull \
    --query appId \
    --output tsv)
```

### Running a Container from ACR in ACI

ACR_LOGINSERVER = $(az acr show --name $ACR_NAME --query loginServer --output tsv)

```
az container create \
    --resource-group psdemo-rg \
    --name psdemo-webapp-cli \
    --dns-name-label psdemo-webapp-cli \
    --port 80 \
    --image $ACR_LOGINSERVER/webappimage:v1 \
    --registry-login-server $ACR_LOGINSERVER \
    --registry-usernanme $SP_APPID \
    --registry-password $SP_PASSWD
```

