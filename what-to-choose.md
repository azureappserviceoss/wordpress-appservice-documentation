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

