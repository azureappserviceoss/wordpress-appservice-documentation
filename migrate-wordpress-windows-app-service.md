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

3.Create a resource group 
    
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

### Add a custom domain

- Purchase a domain on [Azure](https://docs.microsoft.com/en-us/azure/app-service/custom-dns-web-site-buydomains-web-app) or elsewhere if you dont have an existing domain for your app
- Add a CNAME record to map your web app endpoint mysite.azurewebsites.net to your custom domain , say example.com 
- Login to Azure portal and go to your web app . Note your application must be using Standard or Premium Pricing tiers in order to add a domain. If the app is on another pricing tier , please change the pricing tier before moving the next step . 
- Click on **Custom Domains setting -> Add hostname**
- Enter the custom domain ,say exmaple.com and click **Validate**  . If the validation is successful , then click OK to complete adding the custom domain to your web app. If the validation is not successfuly , check if your CNAME record is configured correctly. 
- Login to Wordpress admin dashboard. Go to General Settings and update Site URL  as per instructions in this [article](https://codex.wordpress.org/Changing_The_Site_URL) 




### Add SSL certficate
- Purchase an SSL certificate for your domain on [Azure](https://docs.microsoft.com/en-us/azure/app-service/web-sites-purchase-ssl-web-site) or elsewhere if you dont have domain validated certificate for your web app. 
- Login to Azure portal and go to your web app. Note your application must be using Standard or Premium Pricing tiers in order to add a domain. If the app is on another pricing tier , please change the pricing tier before moving the next step .
- Click on **SSL bindings-> Add binding**
- Upload a certificate as shown in [this article] if you are bringing your own certificate(https://docs.microsoft.com/en-us/azure/app-service/app-service-web-tutorial-custom-ssl#bind-your-ssl-certificate#upload-your-ssl-certificate)
- If you are using an App Service certiifcate , then import your certificate as shown in [this article](https://blogs.msdn.microsoft.com/benjaminperkins/2017/04/12/how-i-configured-an-app-service-certificate-for-my-azure-app-service/)


You can enforce HTTPS for your web app without chanigng web.config  , [learn more here](https://docs.microsoft.com/en-us/azure/app-service/app-service-web-tutorial-custom-ssl#enforce-https)

### Enable Email 
SMTP is not supported in App Service . Hence you need a Email service such as Sendgrid to be confugured with your web app. Find various email services listed here in Azure marketpalce : https://azuremarketplace.microsoft.com/en-us/marketplace/apps?search=email  

Use WordPress plugin to enable SMTP with the type of email service you have selected. For example if using Sendgrid , you can choose to use [Sendgrid WordPress plugin](https://wordpress.org/plugins/sendgrid-email-delivery-simplified/) to configure you app and start using Email functionality . Here is another plugin , you can using if using other email provides like Office 365 , Gmail etc  https://wordpress.org/plugins/wp-email-smtp/ 

### Backup your wordpress app
DO NOT USE Wordpress plugins to backup your web app.  Follow the instructions here on [how to backup you web app](https://docs.microsoft.com/en-us/azure/app-service/web-sites-backup)
- If you are using MySQL in-app database , the database will also be backed up for your web app 
- If you are using Azure database for MySQL, use the backup options for this service available to you https://docs.microsoft.com/en-us/azure/mysql/howto-restore-server-cli#set-backup-configuration . In this case, DO NOT USE database back up as part of Web App backup feature. 

*As best practice we recommend to automate backup your wordpress app and database once every day at minimum in case you need to restore the app*

## Clean up or Deleting web app 
- Login to Azure Portal. 
- Check if all the resources (web app , mysql server and database etc. ) are all in the same resource group within the same subscription 
- Select the resource group and click on DELETE . You will be asked to confirm the delete operation. 
- If the azure resources are across multiple subscriptions , you would need to manually go to each subscription to clean up and delete the resources.
















