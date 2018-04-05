# TroubleShooting

## Configuration 

#### How to resolve Memory Size Exhausted error ?
   The default limit is 128MB. You can increase the Memory limit either using
   1. by adding this to wp-config.php:
   ```define('WP_MEMORY_LIMIT', '128M');```
   2. OR by adding .user.ini at the site root (D:\home\site\wwwroot) with:
   ```memory_limit=256M```

   Now restart your web app to reflect the changes.

#### How to enable advanced WordPress Debugging?
   WordPress provides various options to debug your WordPress site. WordPress uses a constant WP_DEBUG to turn ON debugging for your WordPress site. By default it is turned OFF in wp-config.php file
   ```define('WP_DEBUG', false);```
   To enable it, you need to set it to TRUE

   ```define('WP_DEBUG', true);```
   Note that debugging should be turned be turned OFF after debugging an issue. For more advanced options for debugging WordPress, see [Debugging in WordPress](http://codex.wordpress.org/Debugging_in_WordPress).

#### How do I configure an Email with WordPress? 
   SMTP is not supported in App Service . Hence you need a Email service such as Sendgrid to be confugured with your web app. 
   - Find various email services listed here in Azure marketpalce : https://azuremarketplace.microsoft.com/en-us/marketplace/apps?search=email
   - Use WordPress plugin to enable SMTP with the type of email service you have selected. 

   We recommend using Sendgrid Email Service with the Sendgrid plugin for WordPress . Check out how to configure sendgrid in WordPress [here](https://sendgrid.com/docs/Integrate/Tutorials/WordPress/sendgrid_wordpress_plugin.html)

#### How do I add a domain to WordPress app ?
   - Buy a domain for your web app or use an existing domain 
   - Add a CNAME record where your domain is hosted to map your web app endpoint mysite.azurewebsites.net to your custom domain , say example.com . 
   - Login to Azure protal and add a hostname binding to your azure web app . See [how to add a hostname to web app](https://docs.microsoft.com/en-us/azure/app-service/app-service-web-tutorial-custom-domain#enable-the-cname-record-mapping-in-the-app)
   - If the validation is successful , then click OK to complete adding the custom domain to your web app. If the validation is not successfuly , check if your CNAME record is configured correctly.
   - Login to Wordpress admin dashboard. Go to General Settings and update Site URL  as per instructions in this [article](https://codex.wordpress.org/Changing_The_Site_URL)

#### How do I use GMAIL with WordPress ?
   You can use this plugin which supports multiple Email providers from Gmail, Yahoo, O365 etc  https://wordpress.org/plugins/wp-email-smtp/  . Follow instructions in this [blog post](http://www.wpbeginner.com/plugins/how-to-send-email-in-wordpress-using-the-gmail-smtp-server/) to setup this plugin to use Google. Note when using other email servers like O365 and Yahoo , may have specifics instructions. Contact plugin support team to get help https://wordpress.org/support/plugin/wp-email-smtp

## Error Connecting to Database 

#### I am using ClearDB database and I cannot connect to the database ?
   Use mysql command line or mysql workbench to check if you can access the ClearDB database remotely. If you not able to connect to the database remotely , contact [ClearDB support](http://w2.cleardb.net/contact/) if the web app is not able to connect to the database. 

      **Connect to remote database using MySQL command line :** Run the following command in a terminal ```mysql --host=localhost --user=myname --password=password mydb``` . Please replace the user, password and database names 

      **Connect to remote database using Workbench :** Follow this article to connect to the ClearDB database https://www.inmotionhosting.com/support/website/database-connections/connect-database-remotely-mysql-workbench 

#### I am using MySQL in-app database . I cannot connect to the database using PHPmyadmin?
   MySQL process may not be running for your web app . Make sure you have [ALWAYS ON](https://docs.microsoft.com/en-us/azure/app-service/web-sites-configure) feature turned on for your web app to avoid the web app being unloaded if it has been idle with no traffic for long time. 

   To troublehshoot this issue , see [resolve MySQL in-app database access via PHPmyadmin](https://blogs.msdn.microsoft.com/appserviceteam/2016/09/08/troubleshooting-faq-for-mysql-in-apppreview/#cannot-connect)

#### I am using Azure MySQL database. I cannot connect to the database ?
   - Check if Azure database for MySQL server has SSL Enabled. If your wordpress app is not using SSL to connect to the MySQL server , disable SSL for Azure MySQL database 
   - Check if the firewall rules are enabled to block Web App from accessing the MySQL server . If there are no firewall rules configured , please add the following rule . For more details , see [how to ada firewaill rules to connect from Azure](https://docs.microsoft.com/en-us/azure/mysql/concepts-firewall-rules#connecting-from-azure)
   Name = AllowAzureIPs
   Start IP = 0.0.0.0
   End IP = 0.0.0.0 
   - Check if you can connect to the database using MySQL Workbench : Follow this article to connect to the Azure Database for MySQL https://www.inmotionhosting.com/support/website/database-connections/connect-database-remotely-mysql-workbench 

####  How to fix' Error Establishing a Database Connection' in WordPress?

You get this error when WordPress is unable to establish a database connection. The following reasons could be the reason why the issue occurs:

      **Database information in wp-config.php file may be incorrect**
      Use MySQL Client to access your database with the information in wp-config.php to verify this.

      **Check if both the front end and back end of your WordPress site are returning the different errors**
      If you are getting a different error on the wp-admin page for instance something like “One or more database tables are unavailable. The database may need to be repaired”, then you need to repair your database. To do this add the following wp-config.php file:

      ```define('WP_ALLOW_REPAIR', true);```
      Once you have done that, you can see the settings by visiting this page: http://www.yoursite.com/wp-admin/maint/repair.php

      The user does not need to be logged in to access this functionality when this define is set. This is because its main intent is to repair a corrupted database, Users can often not login when the database is corrupt. So once you are done repairing and optimizing your database, make sure to remove this from your wp-config.php.

      **wp-config.php file may be corrupted**
      Retouch wp-config.php file so that the last modified time stamp for the file is changed.

      **If this error occurs intermittently when your site is on high user load** 
      You may be hitting out outbound port limits at high traffic . Autoscale may not help or work in such cases . Use persistent database connnections for your WordPress app. Note that by default WordPress does not use persistent connections. Contact [WordPress support](https://wordpress.org/support/) on how to use persistent connections. You can fork and update an old plugin for persistent connections to work with your web app https://wordpress.org/plugins/persistent-database-connection-updater/ 

## Manage my WordPress database 

#### I am using MySQL in-app database , how can I manage my database? 
   We provide PHPmyadmin to access the MySQL in-app database. note this database cannot be accessed remotely as this is local to instance on which your app is running. 
   See [how to manage MySQL in-app database](https://blogs.msdn.microsoft.com/appserviceteam/2016/08/18/announcing-mysql-in-app-preview-for-web-apps/#manage-mysqlinapp)

#### I am using Azure MySQL or ClearDB . How do I access my database and run queries ? 
   Use MySQL workbench or MySQL command line utilities to connect and manage your database. 

## Availability Issues : WordPress app is down 

#### My WordPress app is down and I am using MySQL in-app ? 
   If you are using MySQL in-app :
   - Check if you can access your database. See [how to manage MySQL in-app database](https://blogs.msdn.microsoft.com/appserviceteam/2016/08/18/announcing-mysql-in-app-preview-for-web-apps/#manage-mysqlinapp) 
   - If no , then your app may be down becaucse if the database not being available. 
   - This can occur if the App service storage is in read only mode. Usually the issue should not last for more than few minutes. If it lasts longer , your database may be in a bad state . In this case , [restore your web app to a deployment slot](https://docs.microsoft.com/en-us/azure/app-service/web-sites-restore) and swap to production site. If you dont have any backup of your web app , you can contact [Azure Support](https://docs.microsoft.com/en-us/azure/azure-supportability/how-to-create-azure-support-request to get a backup copy of the site and MySQL in-app database 

#### My WordPress app is down ? 
   1. Check if you see wordpress admin login page by going to http://yoursite.azurewebsites.net/wp-admin 
      - If you see a login page , this means only the front end of the site is impacted . This could be due to a plugin or theme . Turn on Wordpress debugging to detect and fix the issue. Check out the step by step guide [here](http://premium.wpmudev.org/blog/learn-how-to-troubleshoot-white-screen-errors-in-wordpress/).

   2. If both front end and admin side of wordpress app are down , check if you can connect to the database. Please see articles above on how to connect to the database . 

   3. Check if App Service platform has any issues by using [App Service diagnostics](Check out the step by step guide [here](http://premium.wpmudev.org/blog/learn-how-to-troubleshoot-white-screen-errors-in-wordpress/)

## Performance Issues : Slow WordPress Site 

#### WordPress app is slow. I am using Free or Shared pricing tier ? 
   For Free and Share pricing tiers , we recommend to use MySQL in-app database to get best performance for you wordpress app . If using ClearDB or any remote database service , you will see some slow performance issues 

#### Wordpress app is slow ? Why ?
   Here are two key design elements in App service that can add some latency to a page request:

   - Web Apps uses remote storage disk 
      - If you are not using MySQL in-app feature , then any other MySQL solution used with Web Apps is not living on the same machine that processes the incoming reuqest 
   - As a result of these architecture design configurations you get the flexibility of all the various features of App Service but if you code makes too many calls to the storage and/or MySQL database per request it can add latency to the page response time.

#### Production Wordpress app  is slow  ? How to improve performance ? 
   1. Make sure database and mysql server are on same region. 
   2. Use the following guidance to improve your performance 
      -  Use and configre WP super cache plugin  https://wordpress.org/plugins/wp-super-cache/ 
      -  Use redis cache to boost database performance with [this  redis plugin](https://wordpress.org/plugins/wp-redis) and Azure redis service 
      - Scale up Web app pricing tier (if you are running on Standard Small for Web App, move to Standard Medium or Large ) 
      - Scale up your database 
      - Compress images using https://wordpress.org/plugins/wp-smushit/ 
      - Add a CDN  in front on your web app 

   If none of these solutions help in resolving the performance issues , this could be due to having too much overhead on your file server or your database. You can profile your web app to detect these issues or choose Azure Virtual machines as an alternate solution to Web Apps. 

## Security 

#### How do I know if my site is under DDoS attack ?
   You will see unusual spikes in your traffic to your web app If you have Application Insights or Google Analytics to track the traffic to your site . If you dont have any analytics on traffic to your site , check if your CPU usage is high . 
   To quickly resolve this you can do enable [auto-scale](https://docs.microsoft.com/en-us/azure/monitoring-and-diagnostics/insights-how-to-scale?toc=%2fazure%2fapp-service%2ftoc.json) for a short period of time to manage the spike in traffic. 

   As best practice , turn on web server logs during this to check which IPs to verify if specific IPs were causing the traffic spike. You can block those IPs using [IP restrictions](https://docs.microsoft.com/en-us/azure/app-service/app-service-ip-restrictions). 

#### My WordPress site is hacked ? What do I do  ? 
   Follow guidance here if your think your site is hacked and you cannot login to your web app https://codex.wordpress.org/FAQ_My_site_was_hacked 

   To avoid this in the future , make sure you harden wordpress app
      1. Add the following in wp-config.php to disbal editing plugin and theme code
      ```
      /* Security for Wordpress : 
      you may wish to disable the plugin or theme editor to prevent overzealous users from being able to edit sensitive files and 
      potentially crash the site. Disabling these also provides an additional layer of security if a hacker gains access to a 
      well-privileged user account.
      Note : If your plugin or theme you use with your app requires editing of the files , comment the line below for 'DISALLOW_FILE_EDIT'
      */
      define('DISALLOW_FILE_EDIT', true);
      ```
      2. Add web.config under "uploads" folder to avoid users form uploading malicious files to the web app See [sample here] (https://github.com/azureappserviceoss/wordpress-azure/blob/master/wp-content/uploads/web.config)
      3. Keep Wordpress version up to date . 
      4. Limit users who have admin access to your WordPress site reducing scope of security risk of human error
      5. Use IP restrictions as needed to protect your app from malicious IPs
      6. Configure a firewall for your application https://docs.microsoft.com/en-us/azure/application-gateway/application-gateway-web-application-firewall-portal 

## Scaling WordPress

### How to Scale a WordPress site ? 
   Make sure your WordPress application ins stateless which means user sessions are stored in the database rather than file system and do not cache anything on the local instance running your app. [Learn how to scale your web app](https://docs.microsoft.com/en-us/azure/app-service/web-sites-scale)

#### How to load balance high traffic WordPress site ?
   For high traffic sites , you can use [auto-scale](https://docs.microsoft.com/en-us/azure/monitoring-and-diagnostics/insights-how-to-scale?toc=%2fazure%2fapp-service%2ftoc.json) with standard pricing tier your get a max 10 instances and with Premium pricing tier you get a max of 20 instances. 

   For geo load balancing , use [Traffic Manager with your web app](https://docs.microsoft.com/en-us/azure/app-service/web-sites-traffic-manager). Before using traffic manager , decide if your database if common to both sites in which setup is easy as per the instructions above. If not , you need to modify your application tp be able to talk to two databases. Contact [WordPress forums](http://wordpress.org/support/) on how to make your app talk to to two databases and keep content in sync. 

### Debugging WordPress 

#### How to resolve White Screen/Blank page?
   If you see a White Screen or a blank page when you access your website, the cause could either be plugin or theme related issue. Try to isolate the theme or plugin causing this issue. Check out the step by step guide [here](http://premium.wpmudev.org/blog/learn-how-to-troubleshoot-white-screen-errors-in-wordpress/).

#### My plugin/theme xxxx is slow or not working ?
   For plugins or themes , you should reach out to the respective theme or plugin support page . You can get additional help from Wordpress forums as well http://wordpress.org/support/ . 

