## Creating Web App in Microsoft Azure


### Using Azure CLI
```
az group create -n webapps-dev-rg -l westus2

az appservice plan create --name webapps-dev-plan \
  --resource-group webapps-dev-rg \
  --sku s1 \
  --is-linux
  
az webapp create -g webapps-dev-rg \
  -p webapps-dev-plan \
  -n mp10344884 \
  --runtime "node|10.14"
```

### Configure Azure Web App Service
1. Secure a web app with SSL
2. Configuring a Database connection String
3. Enabling Diagnostic logging
4. Deploying Code to App Service Web Apps

### Secure a web app with SSL
1. Map custom domain to an azure static web application
2. Map SSL certificate to a domain 
3. Use Basic, Standard, Premium or Isolated plan types (Free tier wont work for SSL certificates)
4. Public vs Private certifactes (Public certificates are commonly used with Public web applications)
5. Managed vs Un-managed certificates 
6. You can enforce HTTPS and TLS versions

### Assosciate a Custom Domain with App Service 
1. Go to Resource Group (Example: There was one created with Domains and DNS)
2. Create a Resource (App Service Domain) -> Creating this will also create a public DNS zone. This lets us map domain records to our app service implementation

