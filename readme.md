# WordPress on Azure App Serice 

WordPress on App Service can be deployed on either Linux or Windows version of App Service.  Azure App Service is great for running a simple Single Instance WordPress site or high traffic multi-Instance WordPress site. 

[App Service Documentation](https://docs.microsoft.com/en-us/azure/app-service/)

## Basics 
To quickly get started with WordPress , use the template on Azure Marketplace . Note these templates are not recommended for critical wordpress applications but are great for personal sites using WordPress.

- WordPress on Windows App Service
- WordPress on Linux App Service

*For running production critical WordPress applications , we recommend to not use Marketplace template but an create an empty web app on App Service and migrate your wordpress app* 

The topic below is focused on setting up Wordpress on **Windows App Service** 

### Understand database options for WordPress 
There are different options on what databases to use with WordPress app.

1. MySQL in-app : MySQL in-app feature enables running MySql natively on Azure App Service platform. This is recommemded if you have a 
    - mostly read only site 
    - need only single instance app service plan 
    - using Windows App Service 

     #### Limitations
    - MySQL currently runs on on a single instance .
    - Enabling [Local cache](https://azure.microsoft.com/en-us/documentation/articles/app-service-local-cache/) is not supported.
    - MySQL database cannot be accessed remotely. You can only access your database content using PHPMyadmin or using MySQL utilities in KUDU debug console. This is described in detail below.
    - Storage is shared between both MySQL and your web app files. Note with Free and Shared plans you may hit our quota limits when using the site based on the actions you perform . Check out [quota limitations](https://azure.microsoft.com/en-us/pricing/details/app-service/plans/) for Free and Shared plans.

    For more details on MySQL in-app feature click [here](https://blogs.msdn.microsoft.com/appserviceteam/2016/08/18/announcing-mysql-in-app-preview-for-web-apps/)

2. Azure database for MySQL :  Azure Database for MySQL is a relational database service based on the open source MySQL Server engine. It is a fully managed database as a service offering capable of handing mission-critical workload with predictable performance and dynamic scalability. For more details on Azure database for MySQL [click here](https://docs.microsoft.com/en-us/azure/mysql/)
3. MySQL on Azure Virutal machines  :  You can choose to run and manage your own MySQL server. This is not covered in details in this documentation .  You can reference [this article](https://docs.microsoft.com/en-us/azure/virtual-machines/windows/classic/mysql-2008r2) for more details 

## Create Local development Environment 
1. Download latest WordPress code from https://wordpress.org 
2. Setup you WordPress environment on IIS web server using the following documentation https://codex.wordpress.org/Installing_on_Microsoft_IIS 
3. Use the sample script below in wp-config.php file to make it easy to switch from local and Azure enviroment wihtout changing the code manually everytime . 
```
//Get the Environment variable AZURE_ENV = true/false if its exists .This variable will be configured in Azure portal for your web app  
$onAzure = getenv ('AZURE_ENV');
if ($appEnv) {
    / Production environment */
    define('DB_HOST', ':/cloudsql/[YOUR_PROJECT_ID]:us-central1:tutorial-sql-instance');
    / The name of the database for WordPress /
    define('DB_NAME', 'tutorialdb');
    / MySQL database username */
    define('DB_USER', 'tutorial-user');
    / MySQL database password /
    define('DB_PASSWORD', 'YOUR_DATABASE_USER_PASSWORD');
} else {
    / Local environment */
    define('DB_HOST', 'localhost');
    / The name of the database for WordPress /
    define('DB_NAME', 'my-local-db');
    / MySQL database username */
    define('DB_USER', 'my-local-db-user');
    / MySQL database password /
    define('DB_PASSWORD', 'YOUR_DATABASE_USER_PASSWORD');
}
```
3. Create a web.config and place it under your site root folder. IIS web server does not recognize .htaccess files and you can use web.config file to manage the server configurations such as URL rewrite rules , caching etc . 
Add the following code in web.config . For more web.config configuration with WordPress , click [here](./webconfig-samples.md) to see options on how to use PHP caching, IP restrictions etc. 

```
<?xmlversion="1.0"encoding="UTF-8"?>
<configuration>
    <system.webServer>
        <staticContent>
            <mimeMap fileExtension="woff" mimeType="application/font-woff" />
            <mimeMap fileExtension="woff2" mimeType="application/font-woff" /> 
         </staticContent>
    </system.webServer>
</configuration> 
```

Now start developing locally and build your WordPress app . Check out WordPress CMS documentation here to start developing your app https://codex.wordpress.org/Getting_Started_with_WordPress 

## Setup Azure environment
Create these azure resources needed for your web app at minimum to host WordPress app 
 
1. Login to [Azure portal](https://portal.azure.com)
2. Create an Empty Web App + MySQL database.[Click here](https://portal.azure.com/#create/Microsoft.WebSiteMySQLDatabase). 
   *For personal sites , you can use [WordPress template](http://portal.azure.com/#create/wordpress.wordpress) in Azure portal*
3. Select your web app and add application setting ```AZURE_ENV=true```. For more details on Application Settings , click [here](https://docs.microsoft.com/en-us/azure/app-service/web-sites-configure#application-settings)

### Deploy your code 

We recommend to deploy your code to staging slot as a best practice rather than to primary web app. [Learn how to create a slot here](https://docs.microsoft.com/en-us/azure/app-service/web-sites-staged-publishing#add-a-deployment-slot).    

You can use the following to deploy to code tp a slot or main web app 
1. [FTP](https://docs.microsoft.com/en-us/azure/app-service/app-service-deploy-ftp)
2. [Local GIT](https://docs.microsoft.com/en-us/azure/app-service/app-service-deploy-local-git) 
3. [Github](https://docs.microsoft.com/en-us/azure/app-service/app-service-continuous-deployment) : You can push your application code to Github repository . Configure Github repository as discused in [this article](https://docs.microsoft.com/en-us/azure/app-service/app-service-continuous-deployment)

  #### Push updates to WordPress code 
Once you have your changes tested locally , its time to deploy the code changes to your web app . Push the app code changes to stage slot [recommended] or primary web app by using :

1. [FTP](https://docs.microsoft.com/en-us/azure/app-service/app-service-deploy-ftp)
2. [Local GIT](https://docs.microsoft.com/en-us/azure/app-service/app-service-deploy-local-git)  : If local git is configured , push the changes using git to the stage slot or primary web app 
3. [Github](https://docs.microsoft.com/en-us/azure/app-service/app-service-continuous-deployment) : If continuous integration is enabled , then once the changes are pushed to Github repository they will be synced to your stage slot or priamry web app based on your configuration. 

### Add a custom domain

- Purchase a domain on [Azure](https://docs.microsoft.com/en-us/azure/app-service/custom-dns-web-site-buydomains-web-app) or elsewhere if you dont have an existing domain for your app
- Add a CNAME record to map your web app endpoint mysite.azurewebsites.net to your custom domain , say example.com 
- Login to Azure portal and go to your web app . Note your application must be using Standard or Premium Pricing tiers in order to add a domain. If the app is on another pricing tier , please change the pricing tier before moving the next step . 
- Click on **Custom Domains setting -> Add hostname**
- Enter the custom domain ,say exmaple.com and click **Validate** 

If the validation is successful , then click OK to complete adding the custom domain to your web app. If the validation is not successfuly , check if your CNAME record is configured correctly. 

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


## Clean up or Deleting web app 
- Login to Azure Portal. 
- Check if all the resources (web app , mysql server and database etc. ) are all in the same resource group within the same subscription 
- Select the resource group and click on DELETE . You will be asked to confirm the delete operation. 
- If the azure resources are across multiple subscriptions , you would need to manually go to each subscription to clean up and delete the resources.













