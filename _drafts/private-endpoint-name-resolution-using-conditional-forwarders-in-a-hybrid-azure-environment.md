---
layout: post
title: 'Private Endpoint Name Resolution: Using Conditional Forwarders in a Hybrid
  Azure Environment'
date: 2024-07-08 12:37 -0400
categories: [Azure]
tags: [azure, domain controllers]     # TAG names should always be lowercase
---

In this article we're going to cover the steps to integrate a custom DNS solution for private endpoint name resolution in a hybrid Azure environment. Microsoft does have documentation for [private endpoint DNS integration](https://learn.microsoft.com/en-us/azure/private-link/private-endpoint-dns-integration), but the hybrid examples on the Microsoft documentation all use Azure Private Resolver for the Azure side of the integration. Azure Private Resolver is great, but it can be redundant and result in unnecessary costs if you've already extended (or plan to extend) your on-prem domain controllers into Azure.

In this example we have the following components in our environment:
- Active Directory domain controllers deployed on-prem and in Azure
- Hybrid network connectivity established between Azure and on-prem using a site-to-site VPN
- A storage account with a private endpoint enabled