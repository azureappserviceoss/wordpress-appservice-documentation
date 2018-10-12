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

## Things to Keep in mind 
- App Service is a platform as a service. We do not update or manage your source code even if the application was created from Azure marketplace using templates such as WordPress, Drupal , Umbraco etc. 
- Customers are required to keep the version of CMS solutions up to date when they use Wordpress, Drupal and other similar apps. 
- Customer must follow [best practices for developing on Azure App Services](https://docs.microsoft.com/en-us/azure/app-service/app-service-best-practices?toc=%2fazure%2fapp-service%2fcontainers%2ftoc.json)
- Application must be optimized for App Service due to its distributed architecture. If you want to migrate as-is , we recommend to choose Virtual machines as a solution

## Basics 
To quickly get started with WordPress , use the template on Azure Marketplace . Note these templates are not recommended for critical wordpress applications but are great for personal sites using WordPress.

- WordPress on Windows App Service
- WordPress on Web app for Containers

*For running production critical WordPress applications , we recommend to not use Marketplace template but an create an empty web app on App Service and migrate your wordpress app* 

[Compare WordPress solution on Windows App Service , Linux App Service and Web app for Containers] (https://blogs.msdn.microsoft.com/appserviceteam/2017/09/12/how-to-for-wordpress-on-app-service-windowslinux/)

## Understand database options for WordPress 
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


## Follow the links based on your needs on next steps
- [Create a WordPress app on Web app for Containers](./create-wordpress-on-web-apppp-for-containers.md)
- [Create a WordPress app on Web app on Windows App Service](./create-wordpress-on-web-app-on-windows.md)
- [Create WordPress app using Multi-Containers on Web app for Containers](https://docs.microsoft.com/en-us/azure/app-service/containers/tutorial-multi-container-app)
- [Migrate WordPress app to Azure] (./migrate-wordpress-windows-app-service.md)

















