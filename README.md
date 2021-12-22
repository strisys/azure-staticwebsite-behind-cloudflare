# Using Azure Static Websites Behind Cloudflare
`December 21st, 2021`

### Table of Contents

1. [Overview](#1)
2. [Azure Static Website Setup](#2)
	- [Creating a Storage Account](#2.1)
	- [Enabling Azure Static Website](#2.2)
	- [Uploading Static Website Assets](#2.3)
3. [Domain Registration & Custom Domains with Azure Static Websites](#3)
4. [Configuring Cloudflare in Front](#4)
	- [Overview](#4.0)
	- [DNS Management Transfer to Cloudflare](#4.1)
	- [Page Rules - Forwarding](#4.2)
	- [Accept Traffic from Cloudflare Only](#4.3)
	- [Transform Rules](#4.4)
5. [Summary](#5)

### <a name='1'>Overview</a>

As part of [Azure blob storage](https://docs.microsoft.com/en-us/azure/storage/blobs/storage-blobs-introduction) exists the capability to host [static websites](https://docs.microsoft.com/en-us/azure/storage/blobs/storage-blob-static-website).  In combination with features from [Cloudflare](https://www.cloudflare.com/), there exists a maintainable, performant, secure, and low cost option for serving static content as this document is meant to prove.  Below are some typical requirements for a company website that consists of only static assets.

- *No public access* via API or directory browsing to website assets
- Assets must be served over HTTPS
- A custom domain is the *only* way to reach the site 
- The proper security headers will be put in place and [tested](https://securityheaders.com/) to meet standards.  
- Assets can be cached and minified *without the need* for a complex build process.

The steps in this page detail that which was performed to set everything up ranging from the purchase of a domain through [GoDaddy](https://www.godaddy.com/) to configuration of both [Azure static websites](https://docs.microsoft.com/en-us/azure/storage/blobs/storage-blob-static-website) and [Cloudflare](https://www.cloudflare.com/) to determine if the specified requirements could be satisfied.  Note that the screenshots used in the hyperlinks in this page where created on `December 21st, 2021` and will deviate from the actual service provider screens over time.

### <a name='2'>Azure Static Website Setup</a>

#### <a name='2.1'>Creating a Storage Account</a>

With an Azure subscription in place, a [storage account was created](https://docs.microsoft.com/en-us/azure/storage/common/storage-account-create?tabs=azure-portal)  [Azure portal](https://portal.azure.com/) which is an obvious required step for using [Azure static websites](https://docs.microsoft.com/en-us/azure/storage/blobs/storage-blob-static-website).  Though there are many ways to create a storage account, the steps below using the [portal](https://portal.azure.com/) were what was used.

1. In the [Azure portal](https://portal.azure.com/), clicked [**+ Create a resource**](./src/images/1.png) in the upper left-hand corner.
2. Entered [`storage account`](./src/images/2.png) in the search box and clicked [**Create**](./src/images/3.png) on the following page.
3. On the [**Basics**](./src/images/4.png) blade entered information like a resource group put this resource in and a *unique* storage account name.  Other options like [redundancy](https://docs.microsoft.com/en-us/azure/storage/common/storage-redundancy) could be changed later so no other configuration was necessary here.

#### <a name='2.2'>Enabling Azure Static Website</a>

Once the storage account was [created](./src/images/6.png), the ability to enable [static websites](https://docs.microsoft.com/en-us/azure/storage/blobs/storage-blob-static-website) was available from the **Static website** blade.  (*NOTE: A [not authorized message on this Static website blade](./src/images/7.png) was be observed requiring some blob access modifications in accordance with [this graphic](./src/images/8.png)*).  Static websites were enabled by simply by [toggling it on](./src/images/9.png), entering the name of index document (e.g. `index.html`), and [choosing Save](./src/images/9.png).  The [URL of static website](./src/images/9.png) was displayed (i.e. `https://staticwebsitecloudflare.z13.web.core.windows.net`) as a result.  This URL was the Azure public website address.

#### <a name='2.3'>Uploading Static Website Assets</a>

Aside from a new public URL, another side-effect of enabling static websites was the creation of a special folder named **$web**. This is the target location for static website assets sourced from this repository's [`src` folder](./src).  To upload files, the **$web** folder was chosen from the [**Storage browser**](./src/images/10.png) blade.  Everything on [this screen](./src/images/11.png) was self-explanatory so the upload mechanics will not be discussed here in detail.  Once all the assets were uploaded, their delivery to a browser were tested *successfully* by navigating to the URL (`https://staticwebsitecloudflare.z13.web.core.windows.net`) specified on the [**Static website**](./src/images/9.png) blade in the last section.  

Although a big success to have a static website up in less than 5 minutes, there was more work to do to satisfy all the requirements listed in the Overview.

### <a name='3'>Domain Registration & Custom Domains with Azure Static Websites</a>

The next goal was to register a custom domain and associate it with the Azure static website created in the last section.  [GoDaddy](https://www.godaddy.com/) was used though any registrar would do.  The [domain registration process on GoDaddy](https://www.godaddy.com/help) was fairly straight-forward so it will *not* be covered in detail here.  The domain chosen for this exercise was [`staticwebsitecloudflare.site`](http://www.staticwebsitecloudflare.site/) and the cost was [`$1.17/year`](./src/images/13.png).  [Auto-renewal](./src/images/14.png) was cancelled since this domain is *only* going to be used for this exercise.

[DNS settings](./src/images/16.png) could be [managed](./src/images/15.png) in the GoDaddy portal.  By default, the [DNS settings](./src/images/17.png) included an `A` record with a special value of `Parked`.  This was a friendly name to the [GoDaddy placeholder page](./src/images/18.png).  One option that came to mind at this point was to change this `A` record value to the [IP for our Azure static website](./src/images/28.png) so the DNS look-ups for [`staticwebsitecloudflare.site`](http://www.staticwebsitecloudflare.site/) would resolve appropriately.  Unfortunately this experiment did *not* work as that IP is *pooled* and Azure requires the FQDN in order to route traffic appropriately.  There was also a `CNAME` record that effectively returned the address of the root domain represented as `@`.  With these default DNS entries in place, the steps and approach chosen to associate our new domain with our Azure static website was as follows.

1. In the GoDaddy DNS settings, changed the `CNAME` record value to the FQDN of the Azure website and set the **Domain name** field to the `www` subdomain address (i.e. `www.staticwebsitecloudflare.site`) on the [**Custom** tab on the **Networking**](./src/images/19.png) blade in the Azure storage account. (*NOTE: The [first option in the Azure instructions eventually worked](./src/images/20.png) but it took a couple of tries.  Its possible the [CNAME record needed time to expire](./src/images/21.png).*).  At this point, navigation to see our website to the `www` subdomain, `www.staticwebsitecloudflare.site`, worked in the browser.   Unfortunately navigation to the root of the domain, `staticwebsitecloudflare.site`, still resolved to the GoDaddy placeholder site.  To remedy this the following steps were performed. 
2. Removed the `A` record in preparation for the next step.
3. Used GoDaddy's [domain forwarding](./src/images/22.png) to forward traffic from `staticwebsitecloudflare.site` to [`www.staticwebsitecloudflare.site`](http://www.staticwebsitecloudflare.site/).  Because of the configuration in Step 1, the assets from the Azure static website were returned in the browser.  

In the end our GoDaddy configuration looked as follows [in this graphic](./src/images/23.png).  GoDaddy added to `A` records for its [forwarding service](https://www.godaddy.com/help/forward-my-domain-with-godaddy-12123) domain forwarding was set up.

At this point, browser navigation to the root domain of [`staticwebsitecloudflare.site`](http://staticwebsitecloudflare.site/) and the subdomain of [`www.staticwebsitecloudflare.site`](http://www.staticwebsitecloudflare.site/) to see the Azure static website both worked.  This was good progress but there were still issues.

1. The [site scored an F](./src/images/24.png) on [securityheaders.io](https://securityheaders.com/). 
2. The content was [*not* being distributed over HTTPS](./src/images/25.png).
3. Browser navigation using the Azure static website address (i.e. `https://staticwebsitecloudflare.z13.web.core.windows.net/`) to see content still worked.
4. Website files were [*not* being compressed](./src/images/26.png) or cached.

The next section introduces Cloudflare to fix these issues.

### <a name='4'>Configuring Cloudflare in Front</a>

#### <a name='4.0'>Overview</a>

Its advantageous to have all the traffic to pass through a "value layer" that provides security, caching, and other features.  Although many cloud providers offer the same features, Cloudflare was easy set up and is [best in class](https://blog.cloudflare.com/cloudflare-traffic) as, the time of this writing, their server handle [10 trillion requests every month](https://blog.cloudflare.com/cloudflare-traffic). 

At a high level, in order for traffic to be *forced* through Cloudflare two things needed to be done.

1. DNS management for the domain must be transferred from GoDaddy to Cloudflare.  
2. In Azure, set the blob service created earlier to accept traffic *only* from Cloudflare servers.  

#### <a name='4.1'>DNS Management Transfer to Cloudflare</a>

With a free account on Cloudflare in place, the following steps were followed to set up Cloudflare as a proxy. 

1. The first step was to [add the site](./src/images/29.png).  It was [at this point, the URL for the custom domain](./src/images/30.png) set up in the last section, was specified and [the free plan as chosen](./src/images/31.png).  From there, Cloudflare [read the GoDaddy DNS records for purposes of import](./src/images/32.png).  During the review only [the CNAME record pointing to our Azure static website domain was kept](./src/images/33.png).  Recall the first two `A` records were for the GoDaddy forwarding service and those were no longer needed.  
2. The next step was to [switch to using Cloudflare's nameservers](./src/images/34.png).  This involved logging back into the GoDaddy portal and [changing the nameservers](./src/images/35.png) to those [provided by Cloudflare](./src/images/34.png).  GoDaddy [issued a warning](./src/images/36.png) for this change.  After making this change *only* [Cloudflare's nameservers](./src/images/37.png) could be seen.  All DNS records were cleared as that was now being managed by Cloudflare.  Also once complete [Cloudflare checked](./src/images/38.png) that the nameservers have been changed successfully.
3. At this point Cloudflare presented some [configuration recommendations](./src/images/39.png) such as [enabling HTTPS and minification](./src/images/40.png) of static website assets. Both [options](./src/images/41.png) were enabled.  It should *not* go unnoticed as these are two significant features being given for free.  Cloudflare is going to manage a valid TLS certificate and minify files thereby reducing effort and cost.

This was it for the *initial setup*.  Shortly after setup (i.e. less than `60 seconds`) an `nslookup` was run to see that Cloudflare's nameservers were being used.

	> nslookup
	...
	
	> set type=NS
	> www.staticwebsitecloudflare.site
	...
	
	staticwebsitecloudflare.site
	        primary name server = cris.ns.cloudflare.com
	        responsible mail addr = dns.cloudflare.com
	        serial  = 2265582889
	        refresh = 10000 (2 hours 46 mins 40 secs)
	        retry   = 2400 (40 mins)
	        expire  = 604800 (7 days)
	        default TTL = 3600 (1 hour)

Browsing to the `www` subdomain of [www.staticwebsitecloudflare.site](https://www.staticwebsitecloudflare.site/) also worked, was [served over HTTPS with a valid certificate](./src/images/43.png), and it was observed that [files were being minified](./src/images/44.png).  

At this point, the outstanding issues were as follows.  

1. The root domain ([staticwebsitecloudflare.site](https://staticwebsitecloudflare.site)) did [not work](./src/images/45.png).
2. Navigation to the [Azure static website address](https://staticwebsitecloudflare.z13.web.core.windows.net) to see the site still worked.
3. The [necessary security headers](./src/images/46.png) to score an acceptable rating on [securityheaders.io](https://securityheaders.com/) were not set.

#### <a name='4.2'>Page Rules - Forwarding</a>

To fix the first issue, what Cloudflare calls [Page Rules](https://developers.cloudflare.com/rules/) were used.  Two rules were put in place that allowed for the [forwarding traffic, originally destined for the root domain, to go to the `www` subdomain](./src/images/47.png).  This was basically the same thing done in GoDaddy.  In the end there were [two redirect rules](./src/images/48.png) for HTTP and HTTPS.  The root domain however still did *not* work and this is because there was no `A` record.  In [Cloudflare DNS settings](./src/images/49.png) it made sense to use the pooled IP address from Azure, which in this case was `104.21.3.7`.  This entry did *not* need to be proxied so the **Proxy state** was set to **DNS only**.
	> nslookup
	...
	
	Non-authoritative answer:
	Name:    staticwebsitecloudflare.site
	Address:  104.21.3.7

Once the specified address was returned from an `nslookup` command, a couple of more steps were necessary and these were somewhat *tricky*.  

1. In the Cloudflare DNS settings, the **Proxy status** for the `CNAME` record was set to **DNS only**.  The was *temporary* and the reason was to re-verify the CNAME record from Azure like was done with GoDaddy.  This would *not* work if the **Proxy status** toggle is set to **Proxied**.  
2. In Azure, the CNAME was re-verified using [Custom tab of the Network blade in the storage account](./src/images/52.png).  The previous entry had to be first cleared, saved, and then reapplied.  
3. While in Azure, HTTP traffic was also allowed using the [Configuration blade and disable Secure transfer required](./src/images/53.png).  The idea was that traffic from Cloudflare is clean and did *not* require encryption.  
4. Finally, the toggle set in Step #1 was reversed so the [**Proxy status** in Cloudflare DNS for the CNAME record was back to proxied](./src/images/54.png).

At this point the following URLs were tested.  Chrome was used with a new Incognito instance with the browner DNS cache cleared for good measure at [chrome://net-internals/#dns](chrome://net-internals/#dns).

- [`http://staticwebsitecloudflare.site`](http://staticwebsitecloudflare.site)
- [`https://staticwebsitecloudflare.site`](https://staticwebsitecloudflare.site)
- [`http://staticwebsitecloudflare.site`](http://staticwebsitecloudflare.site)
- [`https://www.staticwebsitecloudflare.site`](https://www.staticwebsitecloudflare.site)

All requests worked and used HTTPS regardless if HTTP was specified.

#### <a name='4.3'>Accept Traffic from Cloudflare Only</a>

At this point, the website could still be reached using the Azure static website URL of [`https://staticwebsitecloudflare.z13.web.core.windows.net/`](https://staticwebsitecloudflare.z13.web.core.windows.net).  This bypassed Cloudflare defeating its purpose.  Fortunately in Azure, on the [**Firewalls and virtual network** tab under the **Networking** blade](./src/images/55.png) for the blob service, a whitelist of accepted IP addresses can be specified.  [Cloudflare publishes their IP addresses](https://www.cloudflare.com/ips/) which were used in this configuration.  Once the Cloudflare IP address restrictions were in place, it was *not* possible [to reach the website](./src/images/56.png) using the Azure static website URL of [`https://staticwebsitecloudflare.z13.web.core.windows.net`](https://staticwebsitecloudflare.z13.web.core.windows.net).  The URLs tested in the last section still still worked after these changes. 

#### <a name='4.4'>Transform Rules</a>

The final problem that needed to be fixed was the [F security score](./src/images/57.png) from [securityheaders.io](https://securityheaders.com/?q=https%3A%2F%2Fstaticwebsitecloudflare.site&followRedirects=on).  Cloudflare's Transform rules allowed for the specification of HTTP headers to returned given certain conditions.  [This graphic](./src/images/58.png) shows that, given an HTTP GET or POST as request method, the following HTTP response headers are configured to be returned as part of the HTTP response.

- `Content-Security-Policy = upgrade-insecure-requests`
- `Permissions-Policy = geolocation=(self), microphone=()`
- `Referrer-Policy = strict-origin-when-cross-origin`
- `Strict-Transport-Security = max-age=2592000`
- `X-Content-Type-Options = nosniff`
- `X-Frame-Options = DENY`

After setting these security header values, the score from [securityheaders.io](https://securityheaders.com/?q=https%3A%2F%2Fstaticwebsitecloudflare.site&followRedirects=on) was [an `A+`](./src/images/59.png).

### <a name='5'>Summary</a>

In the Overview, the original requirements were as follows.

- *No public access* via API or directory browsing to website static assets
- Assets must be served over HTTPS
- A custom domain is the only way to reach the site 
- The proper security headers will be put in place and tested using [securityheaders.io](https://securityheaders.com/).  The required score is `A+`.
- Assets can be cached and minified *without the need* for a complex build process.

All these were satisfied including caching which has *not* yet been discussed.  Cloudflare has a [caching configuration](./src/images/60.png) that [optimizes delivery of website files](https://developers.cloudflare.com/cache) to the target.  When debugging or deploying a site, however, it might be best to [disable caching on the **Overview**](./src/images/61.png) blade to ensure updated assets are returned and not the cached versions.  

In summary, the combination of [Azure Static Websites](https://docs.microsoft.com/en-us/azure/storage/blobs/storage-blob-static-website) and [Cloudflare](https://www.cloudflare.com/) are a great choice meet the requirements for most static websites.  Of course, DevOps can be put in place as well that will allow for assets to be deployed given a push to a Git repository, for example, but that was beyond the scope of this investigation.