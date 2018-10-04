## What to choose for my WordPress App ? 
If you are looking to migrate existing or new Wordpress app to Azure and not sure which service to choose as there are many options to do the same. 

There are various options as show below and listed below each service are reasons on when to pick that product for your wordpress app

|Azure Virtual Machines|App Service|
|---------|---------|
|VM managed by you|VM managed by the service|
Migrate code as-is| Code would need to be optimized for the this service|Code would need to be optimized for the this service|
|Faster Page load time |
|Manual deployment or Use custom tools| Continuous code deployment available such as Git , VSTS, BitBucket etc |
|Manual setup of Scaling via Scalability Sets| One-click Auto-scaling available with configurable rules for scaling |
|Manual setup of development environments |Use deployment slots for quick setup of Dev, Test , QA environments with easy roll back of change if needed.|
| - | A/B testing using Testing in production feature|
|- |Easy configuration of SSL and Domain to web app|
| - |Authentication for using various identity providers like Azure AD, Google , Microsoft Account etc |
| - |Smart diagnostics via App Service Diagnostics avaiable to debug issues|

