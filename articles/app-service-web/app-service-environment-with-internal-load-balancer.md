---
title: Creating and using an Internal Load Balancer with an App Service Environment | Microsoft Docs
description: Creating and using an ASE with an ILB
services: app-service
documentationcenter: ''
author: ccompy
manager: stefsch
editor: ''

ms.assetid: ad9a1e00-d5e5-413e-be47-e21e5b285dbf
ms.service: app-service
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/11/2017
ms.author: ccompy

---
# Using an Internal Load Balancer with an App Service Environment

> [!NOTE] 
> This article is about the App Service Environment v1. There is a newer version of the App Service Environment that is easier to use and runs on more powerful infrastructure. To learn more about the new version start with the [Introduction to the App Service Environment](../app-service/app-service-environment/intro.md).
>

The App Service Environment (ASE) feature is a Premium service option of Azure App Service that delivers an enhanced configuration capability that is not available in the multi-tenant stamps. The ASE feature essentially deploys the Azure App Service in your Azure Virtual Network(VNet). To gain a greater understanding of the capabilities offered by App Service Environments read the [What is an App Service Environment][WhatisASE] documentation. If you don't know the benefits of operating in a VNet read the [Azure Virtual Network FAQ][virtualnetwork]. 

## Overview
An ASE can be deployed with an internet accessible endpoint or with an IP address in your VNet. In order to set the IP address to a VNet address you need to deploy your ASE with an Internal Load Balancer(ILB). When your ASE is configured with an ILB you provide:

* your own domain or subdomain. To make it easy, this document assumes subdomain but you can configure it either way. 
* the certificate used for HTTPS
* DNS management for your subdomain. 

In return, you can do things such as:

* host intranet applications, like line of business applications, securely in the cloud which you access through a Site to Site or ExpressRoute VPN
* host apps in the cloud that are not listed in public DNS servers
* create internet isolated backend apps which your front end apps can securely integrate with

#### Disabled functionality
There are some things that you cannot do when using an ILB ASE. Those things include:

* using IPSSL
* assigning IP addresses to specific apps
* buying and using a certificate with an app through the portal. You can of course still obtain certificates directly with a Certificate Authority and use it with your apps, just not through the Azure portal.

## Creating an ILB ASE
Creating an ILB ASE is not much different from creating an ASE normally. For a deeper discussion on creating an ASE read [How to Create an App Service Environment][HowtoCreateASE]. The process to create an ILB ASE is the same between creating a VNet during ASE creation or selecting a pre-existing VNet. To create an ILB ASE: 

1. In the Azure portal select **New -> Web + Mobile -> App Service Environment**
2. Select your subscription
3. Select or create a resource group
4. Select or create a VNet
5. Create a subnet if selecting a VNet
6. Select **Virtual Network/Location -> VNet Configuration** and set the VIP Type to Internal
7. Provide subdomain name (this will be the subdomain used for apps created in this ASE)
8. Select Ok and then Create

![][1]

Within the Virtual Network blade there is a VNet Configuration option. This lets you select between an External VIP or Internal VIP. The default is External. If you have it set to External then your ASE will use an internet accessible VIP. If you select Internal, your ASE will be configured with an ILB on an IP address within your VNet. 

After selecting Internal, the ability to add more IP addresses to your ASE is removed and instead you need to provide the subdomain of the ASE. In an ASE with an External VIP the name of the ASE is used in the subdomain for apps created in that ASE. 
If your ASE was called ***contosotest*** and your app in that ASE was called ***mytest*** then the subdomain would be of the format ***contosotest.p.azurewebsites.net*** and the URL for that app would be ***mytest.contosotest.p.azurewebsites.net***. 
If you set the VIP Type to Internal, your ASE name is not used in the subdomain for the ASE. You specify the subdomain explicitly. If your subdomain was ***contoso.corp.net*** and you made an app in that ASE named ***timereporting*** then the URL for that app would be ***timereporting.contoso.corp.net***.

## Apps in an ILB ASE
Creating an app in an ILB ASE is the same as creating an app in an ASE normally. 

1. In the Azure portal select **New -> Web + Mobile -> Web** or **Mobile** or **API App**
2. Enter name of app
3. Select subscription
4. Select or create resource group
5. Select or create App Service Plan(ASP). If creating a new ASP then select your ASE as the location and select the worker pool you want your ASP to be created in. When you create the ASP you select your ASE as the location and the worker pool. When you specify the name of the app you will see that the subdomain under your app name is replaced by the subdomain for your ASE. 
6. Select Create. You should select the **Pin to dashboard** checkbox if you want the app to show up on your dashboard. 

![][2]

Under the app name the subdomain name gets updated to reflect the subdomain of your ASE. 

## Post ILB ASE creation validation
An ILB ASE is slightly different than the non-ILB ASE. As already noted you need to manage your own DNS and you also have to provide your own certificate for HTTPS connections. 

After you create your ASE you will notice that the subdomain shows the subdomain you specified and there is a new item in the **Setting** menu called **ILB Certificate**. The ASE is created with a self-signed certificate which makes it easier to test HTTPS. The portal will tell you that you need to provide your own certificate for HTTPS but this is to drive you to have a certificate that goes with your subdomain. 

![][3]

If you are simply trying things out and don't know how to create a certificate, you can use the IIS MMC console application to create a self signed certificate. Once it is created you can export it as a .pfx file and then upload it in the ILB Certificate UI. When you access a site secured with a self-signed certificate, your browser will give you a warning that the site you are accessing is not secure due to the inability to validate the certificate. If you want to avoid that warning you need a properly signed certificate that matches your subdomain and has a chain of trust that is recognized by your browser.

![][6]

If you want to try the flow with your own certificates and test both HTTP and HTTPS access to your ASE:

1. Go to ASE UI after ASE is created **ASE -> Settings -> ILB Certificates**
2. Set ILB certificate by selecting certificate pfx file and provide password. This step takes a little while to process and the message that a scaling operation is in progress will be shown.
3. Get the ILB address for your ASE (**ASE -> Properties -> Virtual IP Address**)
4. Create a web app in ASE after creation 
5. Create a VM if you don't have one in that VNET (Not in the same subnet as the ASE or things break)
6. Set DNS for your subdomain. You can use a wildcard with your subdomain in your DNS or if you want to do some simple tests, edit the hosts file on your VM to set web app name to VIP IP address. If your ASE had the subdomain name .ilbase.com and you made the web app mytestapp so that it would be addressed at mytestapp.ilbase.com then set that in your hosts file. (On Windows the hosts file is at C:\Windows\System32\drivers\etc\ )
7. Use a browser on that VM and go to http://mytestapp.ilbase.com (or whatever your web app name is with your subdomain)
8. Use a browser on that VM and go to https://mytestapp.ilbase.com You will have to accept the lack of security if using a self-signed certificate. 

The IP address for your ILB is listed in your Properties as the Virtual IP Address

![][4]

## Using an ILB ASE
#### Network Security Groups
An ILB ASE enables network isolation for your apps as the apps are not accessible or even known by the internet. This is excellent for hosting intranet sites such as line of business applications. When you need to restrict access even further you can still use Network Security Groups(NSGs) to control access at the network level. 

If you wish to use NSGs to further restrict access then you need to make sure you do not break the communication that the ASE needs in order to operate. Even though the HTTP/HTTPS access is only through the ILB used by the ASE the ASE still depends on resource outside of the VNet. To see what network access is still required look at the information in the document on [Controlling Inbound Traffic to an App Service Environment][ControlInbound] and the document on [Network Configuration Details for App Service Environments with ExpressRoute][ExpressRoute]. 

To configure your NSGs you need to know the IP address that is used by Azure to manage your ASE. That IP address is also the outbound IP address from your ASE if it makes internet requests. The outbound IP address for your ASE will remain static for the life of your ASE. If you delete and recreate your ASE, you will get a new IP address. To find this IP address go to **Settings -> Properties** and find the **Outbound IP Address**. 

![][5]

#### General ILB ASE management
Managing an ILB ASE is largely the same as managing an ASE normally. You need to scale up your worker pools to host more ASP instances and scale up your Front End servers to handle increased amounts of HTTP/HTTPS traffic. For general information on managing the configuration of an ASE, read the document on [Configuring an App Service Environment][ASEConfig]. 

The additional management items are certificate management and DNS management. You need to obtain and upload the certificate used for HTTPS after ILB ASE creation and replace it before it expires. Because Azure owns the base domain we can provide certificates for ASEs with an External VIP. Since the subdomain used by an ILB ASE can be anything, you need to provide your own certificate for HTTPS. 

#### DNS Configuration
When using an External VIP the DNS is managed by Azure. Any app created in your ASE is automatically added to Azure DNS which is a public DNS. In an ILB ASE you have to manage your own DNS. For a given subdomain such as contoso.corp.net you need to create DNS A records that point to your ILB address for:

    * 
    *.scm 
    ftp 
    publish 


## Getting started
All articles and How-To's for App Service Environments are available in the [README for Application Service Environments](../app-service/app-service-app-service-environments-readme.md).

To get started with App Service Environments, see [Introduction to App Service Environments][WhatisASE]

For more information about the Azure App Service platform, see [Azure App Service][AzureAppService].

[!INCLUDE [app-service-web-whats-changed](../../includes/app-service-web-whats-changed.md)]

[!INCLUDE [app-service-web-try-app-service](../../includes/app-service-web-try-app-service.md)]

<!--Image references-->
[1]: ./media/app-service-environment-with-internal-load-balancer/ilbase-createilbase.png
[2]: ./media/app-service-environment-with-internal-load-balancer/ilbase-createapp.png
[3]: ./media/app-service-environment-with-internal-load-balancer/ilbase-newase.png
[4]: ./media/app-service-environment-with-internal-load-balancer/ilbase-vip.png
[5]: ./media/app-service-environment-with-internal-load-balancer/ilbase-externalvip.png
[6]: ./media/app-service-environment-with-internal-load-balancer/ilbase-ilbcertificate.png

<!--Links-->
[WhatisASE]: http://azure.microsoft.com/documentation/articles/app-service-app-service-environment-intro/
[HowtoCreateASE]: http://azure.microsoft.com/documentation/articles/app-service-web-how-to-create-an-app-service-environment/
[ControlInbound]: http://azure.microsoft.com/documentation/articles/app-service-app-service-environment-control-inbound-traffic/
[virtualnetwork]: https://azure.microsoft.com/documentation/articles/virtual-networks-faq/
[AppServicePricing]: http://azure.microsoft.com/pricing/details/app-service/
[AzureAppService]: http://azure.microsoft.com/documentation/articles/app-service-value-prop-what-is/
[ASEAutoscale]: http://azure.microsoft.com/documentation/articles/app-service-environment-auto-scale/
[ExpressRoute]: http://azure.microsoft.com/documentation/articles/app-service-app-service-environment-network-configuration-expressroute/
[vnetnsgs]: http://azure.microsoft.com/documentation/articles/virtual-networks-nsg/
[ASEConfig]: http://azure.microsoft.com/documentation/articles/app-service-web-configure-an-app-service-environment/
