# Migrate your WordPress app 

## Basics

You can migrate your WordPress app to Windows or Linux App Service web apps. Migrations strategy can be done either by :
1. Using a plugin to export and import your web app 
2. Manual migrations of files and database 

For mission-critical Wordpress applications, I recommend using manual migration strategy due to complexities involved in your application needs. 
For smaller sites , your can use plugins such as [All-in-one WP migration](https://wordpress.org/plugins/all-in-one-wp-migration/) and many other plugins available in Wordpress plugin repository. 

## Manual Migration 

Most apps are manually migrated and it is more safer to do a manual migration , hence this document focuses only on manual migration. Before you start the migration process , backup your entire site and database from your current infrastructure. 

The steps below use Azure CLI to perform azure related operations. Please complete the following before following the steps below
1. Install [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest)
2. If you have multiple subscriptions, please use the following to choose the default subscription as the Azure Pass provided to you 
    
    az account set --subscription my-subscription-name 

3. Create a resource group 
    
    # Use the[az appservice list-locations](https://docs.microsoft.com/en-us/cli/azure/appservice?view=azure-cli-latest#list-locations) Azure CLI command to list available locations. 
    # Create a resource group for web app and database
    az group create --name myResourceGroup --location "West US"    
   
### Create a empty web app 

#### Step 1 : Create an app service plan with the right Sku for your web app. For details , see [here](https://azure.microsoft.com/en-us/pricing/details/app-service/linux/)
  
       az appservice plan create --name myAppServicePlan --resource-group myResourceGroup --sku S1 --is-linux

#### Step 2 :Create a web app. Give it a unique name. Specify any runtime (it will be replaced later)

    az webapp create --name <app_name> --resource-group myResourceGroup --plan myAppServicePlan --deployment-container-image-name <your-docker-user-name>/wordpress-app:latest

### Create a MySQL database 
#### Step 1 : Create a MySQL server using [Azure Database for MySQL](https://azure.microsoft.com/en-us/services/mysql/) . 

In this example , we are creating a MySQL 5.7 server in West US named ```mydemoserver``` in your resource group ```myresourcegroup``` with server admin login ```myadmin```. This is a Gen 4 General Purpose server with 2 vCores. Substitute the ```<server_admin_password>``` with your own value.

	az mysql server create --resource-group myresourcegroup --name mydemoserver  --location westus --admin-user myadmin --admin-password <server_admin_password> --sku-name GP_Gen4_2 --version 5.7

#### Step 2 : Configure Firewall to allow on Azure Services to have access to your MySQL Server. 

	az mysql server firewall-rule create --resource-group myresourcegroup --server mydemoserver --name AllowMyIP --start-ip-address 0.0.0.0 --end-ip-address 0.0.0.0

To create the database , make sure you allow your IP address to access the server where ```<My-IP-Address>``` will be your IP address

	az mysql server firewall-rule create --resource-group myresourcegroup --server mydemoserver --name AllowMyIP --start-ip-address <My-IP-Address> --end-ip-address <My-IP-Address>

#### Step 3 : Disable SSL for MySQL server for the purpose of this lab. By default SSL is enabled on MySQL server . If your app code is connecting to MySQL server via SSL , please skip this step.

	az mysql server update --resource-group myresourcegroup --name mydemoserver --ssl-enforcement Disabled

#### Step 4:  Create a database 

Connect to your MySQL server using MySQL command line utility. Make sure MySQL is installed on your local machine 

	mysql> mysql -h mydemoserver.mysql.database.azure.com -u myadmin@mydemoserver -p	
	mysql>CREATE DATABASE wordpress-db;

#### Step 5: Get Connection information to add it to your wordpress app's wp-config.php

	az mysql server show --resource-group myresourcegroup --name mydemoserver

### Edit wp-config.php to point to new database
Edit wp-config.php to store the database information of Azure MySQL database created in the above step. Follow the guidance [here](https://codex.wordpress.org/Editing_wp-config.php)

### Deploy Wordpress files to web app
To ease of deployment , most users are familiar with FTP . Deploy your files via FTP as described in this [article](https://docs.microsoft.com/en-us/azure/app-service/app-service-deploy-ftp). Once you are more familair with App Service , you can try other deployment options such as [Deploy Continuously](https://docs.microsoft.com/en-us/azure/app-service/app-service-continuous-deployment) or Deploy with [cloud sync](https://docs.microsoft.com/en-us/azure/app-service/app-service-deploy-content-sync).

### Import database to Azure MySQL database 
Import the backup of the database to Azure MySQL . See how to import database to [Azure MySQL ](https://docs.microsoft.com/en-us/azure/mysql/concepts-migrate-import-export)

### Update URLs in Wordpress database in Azure 
Wordpress stores URLs in the database. If you existing database has URLs with custom domain , say www.mydomain.com then you need to overwrite these URLs in the database to test the app in Azure before making your app in Azure ready for production use. See [how to update URLs in wordpress ](https://codex.wordpress.org/Moving_WordPress#Changing_Your_Domain_Name_and_URLs)

### Performance 
Browse your app and test the performance of your app. If page load time does not satisfy your needs , you may add an Azure Redis Service , Azure CDN service and optimize your plugins/code to improve the performance. If you dont want to add these options , I recommend to move the app to Azure Virtual machine instead of App Service. See [What service to choose for WordPress](https://raw.githubusercontent.com/azureappserviceoss/wordpress-appservice-documentation/master/what-to-choose.md)

### Add custom domain 
Once you have the app running successfuly and ready to go live , add the custom domain to Azure web app. See [how to add a domain to web app ](https://docs.microsoft.com/en-us/azure/app-service/app-service-web-tutorial-custom-domain)


















