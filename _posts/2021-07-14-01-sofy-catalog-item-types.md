---
layout: home
title: SoFy Catalog Item Types
description: Understanding the various types of content available in SoFy.
---

## SoFy Catalog Item Types

The SoFy catalog is comprised of five different types of content.

-	Products
-	Modules
-	Demo Packs
-	Demo Docs
-	Proxies

Of these, Products, Modules, and Demo Packs can be deployed in either the standard full-featured mode, or in "Hybrid Mode," which restricts some capabilities for non-HCL users.

#### Products
Products are actual HCL Software offerings, such as BigFix, Commerce, DX, and Domino.

#### Modules
Modules are also called services, microservices, or APIs. These are helper components which generally work in conjunction with one or more products.

#### Demo Packs
Demo Packs are used to create demonstration environments where one or more products are integrated and configured for specific purposes. For instance, a Demo Pack for HCL Commerce might include additional database content, graphics, and integration with OneDB.

#### Demo Docs and Proxies
Some products, such as HCL Accelerate, include built-in demonstrations (and, therefore, don’t require a separate Demo Pack). Demo Docs are a way to provide detailed instructions on how those demos should be configured and presented. 

Similarly, a few HCL products aren’t yet ready for inclusion in SoFy but have external web or demo sites. Proxy entries provide a way for those products to be included in the SoFy Catalog along with some level of documentation and links to their external sites.

#### Hybrid Mode
Hybrid entries are SoFy Products, Modules, or Demo Packs that can be deployed to a SoFy sandbox by anyone, but downloading of the Helm charts for deployment outside a SoFy sandbox is restricted to HCL internal users only.
