# Configuring Cloudflare in front of your Nable instance

Credit:

All credit goes to the work done by /u/Hefty-Hovercraft: https://www.reddit.com/r/msp/comments/g3iwig/how_we_used_a_free_cloudflare_plan_to_hide_our/

Also:

* /u/square-mulberry8261
* /u/m9832
* /u/likwid9
* various members of the https://n-central-community.slack.com/ team for tweaks, improvements, and 2021.1+ support.

Note: This is confirmed working on self-hosted instances only at this time.  This guide will focus on 2021.1 HF5, but should work on previous versions if versiopn specific  steps are configured.

This is using the free tier from Cloudflare - there is no cost to implement this.  There are benefits of going with a paid plan, including additional insite CNAME redirect, WAF, etc., but this guide will not touch on those features. 


This guide will utilize a new domain (referred to as CF Domain here) that will become where you point agents and WebUI users.  It is possible to keep your existing domain/subdomain name and utilize Cloudflare, but that will not be covered here.


## 1. Aquire a domain name and configure Cloudflare's nameservers (NS)

* See https://support.cloudflare.com/hc/en-us/articles/205195708-Changing-your-domain-nameservers-to-Cloudflare

Once your domain has been updated with Cloudflare's nameservers, log into your Cloudflare admin portal at https://dash.cloudflare.com/ and go to the DNS tab.  Configure an A record to point to your Nable server IP.  If possible, use a different IP that your existing Nable hostname resolves to.

You should see something similar to this:

![image](https://user-images.githubusercontent.com/1140952/126249443-1393d8c5-2c13-4591-986a-172e0f72a1a0.png)

## 2. Configure NAT rules.

If you used a new IP for your CF Domain, open port 80 and 443 to your Nable server.  Allow ONLY Cloudflare IPs to connect via this IP.  https://www.cloudflare.com/ips/

Skip this step for now if you will be using your existing IP.

## 3. SSL/TLS

In the Cloudflare admin portal, under SSL/TLS -> Overview, be sure encryption mode is set to Full

![image](https://user-images.githubusercontent.com/1140952/126249879-dc873222-2d98-4f7a-b711-87db65fb6d85.png)

Under SSL/TLS -> Edge Certificate:

* Enabled Always Use HTTPS
* Set Minimum TLS Version to 1.2
* Enable TLS 1.3
* Automatic HTTPS Rewrites


![image](https://user-images.githubusercontent.com/1140952/126250199-9a727f77-c1cf-49da-bf6f-7011167cba27.png)


## 4. Firewall

Go to Firewall -> Firewall Rules

* Rule 1:
   * Name: Block Known Bots
   * When incoming requests match… "Known Bots" equals "On"
   * Then: Block
   * ![image](https://user-images.githubusercontent.com/1140952/126250646-cfb98c90-fedf-4762-942a-4877a8aa14ba.png)
* Rule 2:
   * Name: Block Country
   * When incoming requests match… "Country" does not equal *countries you operate in*
   * Then: Block
   * ![image](https://user-images.githubusercontent.com/1140952/126250780-6fbd0c08-5dc3-48dd-9236-9f06f91915f1.png)
* Rule 3:
   * Name: Agent & Probe Traffic to NC
   * When: Use edit expression and paste the following: ```(http.request.uri.path eq "/dms2/services2/ServerMMS2" and http.user_agent eq "Agent-Probe" and http.request.method eq "POST") or (http.request.uri.path eq "/bosh/bosh/" and http.user_agent eq "" and http.request.method eq "POST") or (http.request.uri.path eq "/dms2/services2/ServerEI2" and http.user_agent eq "Mozilla/5.0 (compatible)" and http.request.method eq "POST") or (http.request.uri.path contains "/images/agent/" and http.user_agent eq "") or (http.request.uri.path contains "agentAssetImageMap.txt") or (http.request.uri.path contains "/download/") or (http.request.uri.path eq "/dms2/services2/ServerII2" and http.user_agent eq "Mozilla/5.0 (compatible)" and http.request.method eq "POST") or (http.request.uri.path eq "/FileTransfer/") or (http.request.uri.path eq "/commandprompt/") or (http.request.uri.path contains "/LogRetrieval") or (http.request.uri.path eq "/dms2/services2/ServerMMS2" and http.user_agent eq "gSOAP/2.8" and http.request.method eq "POST") or (http.request.uri.path eq "/dms2/services2/ServerEI2" and http.user_agent contains "MSP%20Anywhere%20Daemon (unknown version)" and http.request.method eq "POST") or (http.request.uri.path eq "/dms2/services2/ServerII2" and http.user_agent contains "MSP%20Anywhere%20Daemon (unknown version)" and http.request.method eq "POST") or (http.request.uri.path eq "/dms2/services2/ServerII2" and http.user_agent eq "CodeGear SOAP 1.3" and http.request.method eq "POST") or (http.request.uri.path eq "/dms2/services2/ServerII2" and http.user_agent eq "Mozilla/4.0 (compatible; MSIE 6.0; MS Web Services Client Protocol 4.0.30319.42000)" and http.request.method eq "POST") or (http.request.uri.path eq "/dms2/hello")```
   * Then: Allow
* Rule 4:
   * Name: WebUI
   * When: Use edit expression and paste the following.  Be sure to replace YOURSONAMEHERE with your Nable SO Name: ```(http.request.uri.path eq "/") or (http.request.uri.path contains "/login/") or (http.request.uri.path contains "/dojoroot/") or (http.request.uri.path contains "/favicon.ico") or (http.request.uri.path contains "/cdn-cgi/access/authorized") or (http.request.uri.path contains "/images/") or (http.request.uri.path contains "/stylesheets/") or (http.request.uri.path contains "/js/") or (http.request.uri.path contains "/angular/") or (http.request.uri.path contains "/fonts/") or (http.request.uri.path contains "/rest/") or (http.request.uri.path eq "/assetDiscoveryEditDeviceAction1.do") or (http.request.uri.path eq "/dms/services/ServerUI") or (http.request.uri.path eq "/dms2/services2/ServerUI2") or (http.request.uri.path eq "/UIFileTransfer") or (http.request.uri.path contains "/missingPatchesReportAction.do") or (http.request.uri.path eq "/so/YOURSONAMEHERE") or (http.request.uri.path eq "/detailedAssetAction.do") or (http.request.uri.path eq "/deepLinkAction.do") or (http.request.uri.path contains "/downloadFileServlet.download") or (http.request.uri.path contains "/configurationSummaryAction.do") or (http.request.uri.path contains "/IndexAction.action") or (http.request.uri.path contains ".action") or (http.request.uri.path contains "/reportAction.do") or (http.request.uri.path contains "/chartRendererAction.do") or (http.request.uri.path contains "/patchInventoryReportAction.do") or (http.request.uri.path contains "/dms/") or (http.request.uri.path eq "/dms2/services2/ServerII2" and http.user_agent contains "Mozilla/4.0 (compatible; MSIE 6.0; MS Web Services Client Protocol" and http.request.method eq "POST") or (http.request.method eq "GET" and http.request.uri.path eq "/login") or (http.request.uri.path eq "/dms2/hello")```
   * Then Allow
* Rule 5:
   * Name: Block everything else
   * When incoming requests match… "URI" does not contain "randomfailstring"
   * Then: Block
   * ![image](https://user-images.githubusercontent.com/1140952/126251775-d94ec32b-2253-46e8-bf54-00329fa482ed.png)


## 5. Access

### Login Methods

Navigate to the Access tab in the Cloudflare admin page.  By default you should have One-Time Pin as an option for Login Methods, but you should add an Identity provider like Azure AD or another SAML compatible IdP.  This will be used to allow your internal team/techs to authenticate with Cloudflare and access your Nable server login page.  This guide will not go into depth on configuring a IdP, pleae see https://developers.cloudflare.com/access/about/quick-start for additional information.

If you require users outside your IdP be able to access Nable (ie Clients who access their machines), you can configure a public IdP like Facebook, or create an Access Group and utilize the One-Time Pin function.  We will touch on this later.

### Access Policies

First, create your access policies to let your users access the WebUI.

#### Main Policy

**For Nable 2020.1 and OLDER:**

* Name: Main Policy
   * Leave the application domain subdomain and path values blank
   * Click ‘Add New Policy’, name it ‘Technicians’ and set to Allow
   * Include ‘Emails Ending in’ @ yourmsp.com (whatever your IdP email suffix is)
   * Configure Session Duration to be 12 hours
   * Save and close the Access Policy


**For Nable 2021.1 and NEWER:**

* Main Policy
   * Name: Leave the application domain subdomain blank, and enter ```login``` for the path
   * Click ‘Add New Policy’, name it ‘Technicians’ and set to Allow
   * Include ‘Emails Ending in’ @ yourmsp.com (whatever your IdP email suffix is)
   * Configure Session Duration to be 12 hours
   * Save and close the Access Policy


#### Access Groups

For both versions of Nable, if you would like to allow specific external email addresses to be able to login sing One-Time PINs:

* Create an Access Group called "Client Access - Nable" or similar
* Include: Emails
   * Enter the email addresses you would like to allow into the group
* Go back to your Main Policy, and in addition to your existing "Technicians" policy, add a new policy called "Client Access - Nable" or similar.
   * Set this Policy to Allow
   * Include the Access Group you just created

This is an example Main Policy for a 2021.1 that is configured to allow both IdP users and an Access Group to access the Nable WebUI:

![image](https://user-images.githubusercontent.com/1140952/126256976-a38f8eec-09d9-4fc3-87b8-9a01c9b443df.png)


#### Access Policies - Bypass

For both versions of Nable, create additional Accesss Policies for each of the following, entering each as the ```path``` of the policy.  The policy should be configured for ```Bypass``` for ```Everyone``` on each.

* /dms/
* /download/
* /bosh/bosh/
* /commandprompt/
* /dms/services/ServerMMS/
* /FileTransfer/
* /LogRetreival/
* /services/ServerMMS/
* /dms2/services2/ServerMMS2/
* /dms/services/ServerUI
* /dms2/services2/ServerUI2
* /dms2/services2/ServerII2/
* /dms2/services2/ServerEI2/
* /images
* /indexIntegrationExternalPageAction.action
* /fonts/
* /images/agent/
* /dms2/hello
* /rest/device-active-incident/
* /rest/dashboard
* /rest/lan-devices
* /tunnel/request.tunnel
* /UIFileTransfer

Here is an example of one of the above paths configured in it's policy:

![image](https://user-images.githubusercontent.com/1140952/126256389-66e76bbf-3f5b-4f05-82eb-f94e37c27fb6.png)


**For Nable 2021.1 and NEWER Only:**

This step is optional, but no loss of functionality has been discovered yet.

Nable 2021.1+ brought a new built-in Envoy proxy to the Nable server, which redirects to /login automatically when accessing the site.  Unfortunately for our use, this also leaves our root CF Domain accessible to scraping, and able to be easily identified as an Nable server.  We can reduce our exposure by enabling a redirect on the page:

* Create a bypass policy for the root of your Application Domain (subdomain and path both blank):
   * ![image](https://user-images.githubusercontent.com/1140952/126255643-3fa1b874-5ba2-4a7e-8bab-8f73ac07f11e.png)
* Navigate to Rules, and create a new Page Rule to redirect the root of your new CF Domain to /login:
   * ![image](https://user-images.githubusercontent.com/1140952/126255970-b1ec9bcd-0f7b-4575-a51b-4aff17ba147e.png)





