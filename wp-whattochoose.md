## What to choose for my WordPress App ? 
If you are looking to migrate existing or new Wordpress app to Azure and not sure which service to choose as there are many options to do the same. 

There are various options as show below and listed below each service are reasons on when to pick that product for your wordpress app

||Windows App Servic|Web app for Containers
|Azure Virtual Machines|Windows App Service|Web app for Conatiners|
|---------|---------|---------|
|VM managed by you|VM managed by the service|VM managed by the Service|
Migrate code as-is| Code would need to be optimized for the this service|Code would need to be optimized for the this service|
|Faster Page load time <td colspan=2>Page load time may be higher than 1sec , code optimization and other caching services such as redis may be needed to get best performance
|Manual deployment or Use custom tools<td colspan=2> Continuous code deployment available such as Git , VSTS, BitBucket etc |
|Manual setup of Scaling via Scalability Sets <td colspan=2> One-click Auto-scaling available with configurable rules for scaling |
|Manual setup of development environments <td colspan=2>  Use deployment slots for quick setup of Dev, Test , QA environments with easy roll back of change if needed.|
| - <td colspan=2> A/B testing using Testing in production feature|
|- <td colspan=2> Easy configuration of SSL and Domain to web app|
| - <td colspan=2> Easy Authentication for using various identity providers like Azure AD, Google , Microsoft Account etc |
| - <td colspan=2> Smart diagnostics via App Service Diagnostics avaiable to debug issues|

