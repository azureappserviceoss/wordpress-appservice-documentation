# Using Web.config 
Web.config is the main configuration file for web application running on IIS web server. It is an XML document that resides in the root directory of the site or application and contains data about how the web application will act.

Web.config can be use to enable/disable MIME types , use server level caching , url redirection etc and lot more. 

## Using Permalinks
"Pretty" permalinks usually require mod_rewrite if not running on IIS. For IIS you can use Microsoft's [URL Rewrite Module](https://www.iis.net/downloads/microsoft/url-rewrite) instead. It does support WordPress's pretty permalinks. Once installed, open the web.config file in the WordPress folder and add the following rule to the system.webServer element

URL rewrite modules is enabled on App Service by default.  

```
<?xml version="1.0" encoding="UTF-8"?>
    <configuration>
    <system.webServer>
        <rewrite>
            <rules>
                <rule name="WordPress Rule" stopProcessing="true">
                    <match url=".*" />
                    <conditions>
                    <add input="{REQUEST_FILENAME}" matchType="IsFile" negate="true" />
                    <add input="{REQUEST_FILENAME}" matchType="IsDirectory" negate="true" />
                    </conditions>
                    <action type="Rewrite" url="index.php" />
                </rule>
            </rules>
        </rewrite>
    </system.webServer>
</configuration>
```

## Redirect to HTTPS
If you want your site to be easily discoverable and more user friendly, you probably would not want to return 403 response to visitors who came over unsecure HTTP connection. Instead you would want to redirect them to the secure equivalent of the URL they have requested. With URL Rewrite Module you can perform this kind of redirection by using the following rule. Include the rule below within ```<rewrite>``` section in the web.config file. See samples above used for Permalinks 

```
<rule name="Redirect to HTTPS" stopProcessing="true">  
<match url="(.*)" />  
<conditions>  
<add input="{HTTPS}" pattern="^OFF$" />  
</conditions>  
<action type="Redirect" url="https://{HTTP_HOST}/{R:1}" redirectType="Permanent" />  
</rule>  
```
## Canonical Hostnames
Very often you may have one IIS web site that uses several different host names. The most common example is when a site can be accessed via http://www.yoursitename.com and via http://yoursitename.com. Or, perhaps, you have recently changed you domain name from oldsitename.com to newsitename.com and you want your visitors to use new domain name when bookmarking links to your site. A very simple redirect rule will take care of that:

```
<rule name="Canonical Host Name" stopProcessing="true">  
<match url="(.*)" />  
<conditions>  
<add input="{HTTP_HOST}" negate="true" pattern="^myhostname\.com$" />  
</conditions>  
<action type="Redirect" url="http://myhostname.com/{R:1}" redirectType="Permanent" />  
</rule>```
  

## PHP Caching 
Static content (such as plain HTML pages, images, style sheets, etc) is, by default, automatically cached by IIS. With the IIS Output Caching module you can cache all pages generated by PHP, vary what is cached by query string parameter value, or vary what is cached by header value. Which of those options you choose to use will depend on your application, which brings up the next question…

The output caching module caches entire pages and when a page is served from cache, none of its PHP code is executed. You can control which pages are cached first by page extension (e.g. .php), then by query string parameter and header type. Use output caching when you have entire pages whose content doesn’t change very often. (What is “very often” will depend on your application. 

For a detailed list of caching scenarios using web.config , see [here](https://blogs.msdn.microsoft.com/brian_swan/2011/06/08/performance-tuning-php-apps-on-windowsiis-with-output-caching/) . 

### Cache files until file is changed or modified 
To create a file change notification cache rule, open (or create) the web.config file in your site’s root directory. Then, add the <caching> element as a child element <system.webServer> element as shown here:

```<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <system.webServer>
       <!-- Adding Cache profile to  cache until a file is modified -->
        <caching>
            <profiles>
                <add extension=".php" 
                     policy="CacheUntilChange" />
            </profiles>
        </caching>
    </system.webServer>
</configuration>
```

### Cache files for a period of time 
If you’d rather that a page be expired from the cache at some time interval (instead of waiting for a change to the source file, as above), add a <caching> element to your web.config file as shown here:
```
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <system.webServer>
        <caching>
        <!-- Adding Cache profile to cache for a time period  -->
            <profiles>
                <add extension=".php" 
                     policy="CacheForTimePeriod" 
                     duration="00:00:30" />
            </profiles>
        </caching>
    </system.webServer>
</configuration>
```

## Adding IP restrictions 
IP Restrictions allow you to define a list of IP addresses that are allowed to access your app. The allow list can include individual IP addresses or a range of IP addresses defined by a subnet mask. When a request to the app is generated from a client, the IP address is evaluated against the allow list. This can be useful during a DDoS attack. 

Follow instructions in [this article](https://docs.microsoft.com/en-us/azure/app-service/app-service-ip-restrictions) to add IP restrictions from Azure portal which automatically modifies the web.config with your restriction rules. 

If you want to use web.config manually to set these rules , see samples below common scenarios . 

Note: The allowable values for denyAction are:
- AbortRequest  (returns an HTTP status code of 0)
- Unauthorized (returns an HTTP status code of 401)
- Forbidden (returns an HTTP status code of 403).  Note this is the default setting.
- NotFound (returns an HTTP status code of 404)



#### Code snippet to to handle concurrent requests:
When using this rule , if a client exceeds the concurrent requests their IP will be blocked  temporary for further request for a short period of time. 

```
<system.webServer>
   <security>
      <dynamicIpSecurity>
         <denyByConcurrentRequests enabled="true" maxConcurrentRequests="10"/>
      </dynamicIpSecurity>
   </security>
</system.webServer>
```


#### Code snippet to to handle maxRequests within a specified time frame:
When using this rule, requests from IP addresse will be blocked when client exceeds max requests configured in the rule over a period of time 

```
<system.webServer>
   <security>
      <dynamicIpSecurity denyAction="NotFound">
         <denyByRequestRate enabled="true" maxRequests="15" requestIntervalInMilliseconds="5000"/>
      </dynamicIpSecurity>
   </security>
</system.webServer>
``` 

### Blocking specific IPs

You can add rules as shown below to allow a specific IP and block other IPs . To learn more about ipSecurity , click [here](https://docs.microsoft.com/en-us/iis/configuration/system.webserver/security/ipsecurity/#configuration)
```
 <security> 
        <ipSecurity allowUnlisted="false"> 
            <clear/> 
             <add ipAddress="127.0.0.1" allowed="true"/>
             <add ipAddress="83.116.19.53" allowed="true"/> 
        </ipSecurity>  
    </security>
```




