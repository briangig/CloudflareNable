# Configuring Cloudflare in front of your Nable instance

Credit:

All credit goes to the work done by /u/Hefty-Hovercraft: https://www.reddit.com/r/msp/comments/g3iwig/how_we_used_a_free_cloudflare_plan_to_hide_our/

Also:

* /u/square-mulberry8261
* /u/m9832
* /u/likwid9
* various members of the https://n-central-community.slack.com/ team

For tweaks, improvements, and 2021.1+ support.

Note: This is confirmed working on self-hosted instances only at this time.  This guide will focus on 2021.1 HF5, but should work on previous versions if specific noted steps are configured.


This guide will utilize a new domain (referred to as CF Domain here) that will become where you point agents and WebUI users.  It is possible to keep your existing domain/subdomain name and utilize Cloudflare, but that will not be covered here.


## 1. Aquire a domain name and configure Cloudflare's nameservers (NS)

* See https://support.cloudflare.com/hc/en-us/articles/205195708-Changing-your-domain-nameservers-to-Cloudflare

Once your name servers have been configured, log into your Cloudflare admin portal and go to the DNS tab.  Configure an A record to point to your Nable server IP.  If possible, use a different IP from your existing Nable hostname.

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
