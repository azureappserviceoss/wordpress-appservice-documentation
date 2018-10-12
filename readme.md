# WordPress on Azure App Serice 

WordPress on App Service can be deployed on either Linux or Windows version of App Service.  Azure App Service is great for running a simple Single Instance WordPress site or high traffic multi-Instance WordPress site. 

[App Service Documentation](https://docs.microsoft.com/en-us/azure/app-service/)

## What to choose for my WordPress App ? 
If you are looking to migrate existing or new Wordpress app to Azure and not sure which service to choose as there are many options to do the same. 

There are various options in [Azure] but here are two common Services selectecd typically by users. The Table below listed below gives you an insight to make the decision on what to choose 

|Azure Virtual Machines|App Service|
|---------|---------|
|VM managed by you|VM managed by the service|
|Code is managed by you | Code is managed by you |
|Migrate code as-is once VM is configured to your needs| Code would need to be optimized for the this service since you have restrcited access to the VM|
|Faster Page load time |Page load times vary based on how app performs on the platform. Code optimizations , and use of caching service like Redis cache , CDN etc will be needed to achieve best results|
|Manual deployment or Use custom tools| Continuous code deployment available such as Git , VSTS, BitBucket etc |
|Manual setup of Scaling via Scalability Sets| One-click Auto-scaling available with configurable rules for scaling |
|Manual setup of development environments |Use deployment slots for quick setup of Dev, Test , QA environments with easy roll back of change if needed.|
| - | A/B testing using Testing in production feature|
|- |Easy configuration of SSL and Domain to web app|
| - |Authentication for using various identity providers like Azure AD, Google , Microsoft Account etc |
| - |Smart diagnostics via App Service Diagnostics avaiable to debug issues|


## Basics 
To quickly get started with WordPress , use the template on Azure Marketplace . Note these templates are not recommended for critical wordpress applications but are great for personal sites using WordPress.

- WordPress on Windows App Service
- WordPress on Linux App Service

*For running production critical WordPress applications , we recommend to not use Marketplace template but an create an empty web app on App Service and migrate your wordpress app* 

### Understand database options for WordPress 
There are different options on what databases to use with WordPress app.

1. **MySQL in-app** : MySQL in-app feature enables running MySql natively on Azure App Service platform. This is recommemded if you have a 
    - mostly read only site 
    - need only single instance app service plan 
    - supports Windows App Service ONLY

     #### Limitations
    - MySQL currently runs on on a single instance .
    - Enabling [Local cache](https://azure.microsoft.com/en-us/documentation/articles/app-service-local-cache/) is not supported.
    - MySQL database cannot be accessed remotely. You can only access your database content using PHPMyadmin or using MySQL utilities in KUDU debug console. This is described in detail below.
    - Storage is shared between both MySQL and your web app files. Note with Free and Shared plans you may hit our quota limits when using the site based on the actions you perform . Check out [quota limitations](https://azure.microsoft.com/en-us/pricing/details/app-service/plans/) for Free and Shared plans.

    For more details on MySQL in-app feature click [here](https://blogs.msdn.microsoft.com/appserviceteam/2016/08/18/announcing-mysql-in-app-preview-for-web-apps/)

2. **Azure database for MySQL**:  Azure Database for MySQL is a relational database service based on the open source MySQL Server engine. It is a fully managed database as a service offering capable of handing mission-critical workload with predictable performance and dynamic scalability. For more details on Azure database for MySQL [click here](https://docs.microsoft.com/en-us/azure/mysql/)
3. **Azure Mariadb** : Database offering on Azure using MariaDB server instaead of native MySQL servers. For more details see [here](https://azure.microsoft.com/en-us/services/mariadb/)
4. **MySQL on Azure Virutal machines**  :  You can choose to run and manage your own MySQL server. This is not covered in details in this documentation .  You can reference [this article](https://docs.microsoft.com/en-us/azure/virtual-machines/windows/classic/mysql-2008r2) for more details 

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















