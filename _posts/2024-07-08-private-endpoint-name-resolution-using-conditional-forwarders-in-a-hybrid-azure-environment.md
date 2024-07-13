---
layout: post
title: 'Private Endpoint Name Resolution: Using Conditional Forwarders in a Hybrid
  Azure Environment'
categories:
- Azure
tags:
- azure
- active directory
- domain controllers
date: 2024-07-08 23:51 -0400
---
In this article we're going to cover the steps to integrate a custom DNS solution for private endpoint name resolution in a hybrid Azure environment. Microsoft has documentation for [private endpoint DNS integration](https://learn.microsoft.com/en-us/azure/private-link/private-endpoint-dns-integration), but the hybrid examples on Microsoft Learn all use Azure Private Resolver for the Azure side of the integration. Azure Private Resolver is great, but it can be redundant and result in extra unnecessary costs if you've already extended (or plan to extend) your on-prem domain controllers into Azure.

## What are private endpoints and what problem are we solving here?
Azure private endpoints provide a way for you to connect to your Azure PaaS resources over a private network. If you're not using a custom domain or your Azure resource doesn't support custom domains, you'll need a way to properly resolve the FQDN of the resource to the new private IP (the private endpoint).

The best way to do this is by using an Azure private DNS zone. However, the private DNS zone alone won't solve your issue if you're using a custom DNS solution. The content below describes the extra steps required to get everything resolving the way it should. The configuration below also serves as a simple split-brain DNS setup so that internal users hit the private endpoint and external users hit the public endpoint all while using the same FQDN.

> Disable the public endpoint on the resource unless you require it to be accessible from the internet.
{: .prompt-tip }

In this example we have the following components in our environment:
- Active Directory domain controllers deployed on-prem and in Azure
- Azure VNET DNS settings configured to point to the Azure domain controllers
- Hybrid network connectivity established between Azure and on-prem using a site-to-site VPN
- A storage account with a [private endpoint enabled](https://learn.microsoft.com/en-us/azure/private-link/tutorial-private-endpoint-storage-portal?tabs=dynamic-ip#create-private-endpoint)

> This example uses a storage account private endpoint for demonstration purposes. The same general steps can be applied to any resource that supports private endpoints, just be sure to use the appropriate private DNS zone name as documented [here](https://learn.microsoft.com/en-us/azure/private-link/private-endpoint-dns#commercial).
{: .prompt-info }

### Link the Private DNS Zone to the appropriate VNET(s)
Azure will create the private DNS zone for you by default when creating a private endpoint. Depending on the specifics of your environment, the private DNS zone may or may not be linked to the correct VNET that contains your domain controllers. The private DNS zone must be linked to the same VNET that contains the domain controllers. In our case the private DNS zone was not linked to the proper VNET upon creation, so we added the link manually after the fact:
![Private DNS Zone Links](custom/vnet-link.png){: width="1084" height="439" }

### Create a conditional forwarder for the Azure-based domain controller
1. Logon to the Azure domain controller and launch DNS manager
2. Right click on conditional forwarders and click create new conditional forwarder.
3. Create the conditional forwarder like shown below. The DNS domain name will be specific to the Azure resource type for which you are configuring a conditional forwarder. In this case, we're creating a forwarder for the file service of an Azure storage account. Enter the [Azure DNS VIP](https://learn.microsoft.com/en-us/azure/virtual-network/what-is-ip-address-168-63-129-16) `168.63.129.16` as the master server:

![Azure conditional forwarder](custom/azure-dc-forwarder.png){: width="788" height="550" }

> Be sure to uncheck the box that says "Store this conditional forwarder in Active Directory, and replicate it as follows:"
{: .prompt-tip }

### Create a conditional forwarder for the on-prem domain controllers
1. Logon to the on-prem domain controller and launch DNS manager.
2. Right click on conditional forwarders and click create new conditional forwarder.
3. Create the conditional forwarder like shown below. The DNS domain name will be specific to the Azure resource type for which you are configuring a conditional forwarder. In this case, we're creating a forwarder for the file service of an Azure storage account. Enter the the IPs of your Azure-based domain controllers. The IP of the Azure-based domain controller in our example is `10.0.0.4`:

> The Azure DNS VIP will not respond to queries outside of an Azure VNET, so the on-prem domain controllers must forward their requests for this domain to the Azure-based domain controllers.
{: .prompt-info }

![On-prem conditional forwarder](custom/on-prem-dc-forwarder.png){: width="801" height="557" }

> Be sure to uncheck the box that says "Store this conditional forwarder in Active Directory, and replicate it as follows:"
{: .prompt-tip }

### Test
Run a ping test to verify your Azure resource is now resolving to its private endpoint address:
![Private Endpoint DNS Test](custom/pvt-endpoint-test.png){: width="742" height="153" }