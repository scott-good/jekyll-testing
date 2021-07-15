
---
layout: home
title: Good Thinking
description: An example of a blog that's half a bubble (or so) off-level.
---
## How to Get Cloud Native Ready on SoFy
- [SoFy Catalog Item Types](#sofy-catalog-item-types)
- [SoFy Catalog Metadata](#sofy-catalog-content-metadata)
  - [Generic Metadata](#generic-metadata)
  - [Demo Packs](#demo-packs-metadata)
  - [Demo Docs & Proxies](#demo-docs-and-proxy-metadata)
  - [Hybrid Mode](#hybrid-mode-metadata)
  - [Incompatibilities](#incompatibilities-metadata)
- [Helm Chart Requirements](#helm-chart-requirements)
  - [Chart Acceptance](#chart-acceptance)
    - [1. Helm 3 Chart](#1-helm-3-chart)
    - [2. Helm Install](#2-helm-install)
    - [3. Self Contained](#3-self-contained)
    - [4. Alias or Name Override for Dependency Charts](#4-alias-or-name-override-for-dependency-charts)
    - [5. Self Healing](#5-self-healing)
    - [6. Upgrade](#6-upgrade)
    - [7. Scaling](#7-scaling)
  - [SoFy Integration](#sofy-integration)
    - [1. FlexNet License](#1-flexnet-license)
      - [Requirements](#requirements)
      - [How to Enable](#how-to-enable)
    - [2. SoFy Solution Routing](#2-sofy-solution-routing)
      - [Default Routing](#default-routing)
      - [Custom Routing and Exposing TCP Ports](#custom-routing-and-exposing-tcp-ports)
    - [3. Global Configs](#3-global-configs)
      - [Chart.yaml file](#chartyaml-file)
      - [values.yaml file](#valuesyaml-file)
      - [Useful global values](#useful-global-values)
    - [4. Solution Console UI Integration](#4-solution-console-ui-integration)
      - [Providing Links to Your End Points](#providing-links-to-your-end-points)
        - [Links Schema](#links-schema)
        - [Links Schema (Deprecating soon)](#links-schema-deprecating-soon)
      - [Disable default public routes and default authentication](#disable-default-public-routes-and-default-authentication)
      - [Expose additional application log files](#expose-additional-application-log-files)
      - [Specify required FlexNet feature names](#specify-required-flexnet-feature-names)
    - [5. Pods Requirements](#5-pods-requirements)
      - [Resource Requests and Limits](#resource-requests-and-limits)
      - [Health Checks](#health-checks)
      - [Labels and Selectors](#labels-and-selectors)
    - [6. Getting Sofy Domain in a Container](#6-getting-sofy-domain-in-a-container)
    - [7. Kubernetes Resource Naming Best Practice](#7-kubernetes-resource-naming-best-practice)
- [Windows Images](#windows-images)
  - [Use Node Selectors and Tolerations](#use-node-selectors-and-tolerations)
  - [Windows Pods Examples](#windows-pods-examples)
- [Tips and Tricks](#tips-and-tricks)
- [Also reference](#also-reference)

**Note**: This document will continue evolving with the HCL software cloud native journey.

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


## SoFy Catalog Content Metadata

To function properly, the SoFy Catalog UI requires metadata formatted and used as described on the following sections. Details are provided for:

-	[Generic metadata](#generic-metadata) (applies to all SoFy catalog items)
-	[Demo Pack metadata](#demo-packs-metadata)
-	[Demo Docs and Proxy metadata](#demo-docs-and-proxy-metadata)
-	[Hybrid Mode metadata](#hybrid-mode-metadata)
-	[Incompatibilities metadata](#incompatibilities-metadata)

SoFy Catalog expects the metadata described below for the catalog UI to function properly with all contents (e.g., content card display, user documentation, API documentation display, catalog filtering, etc.). The majority of this information comes from a new metadata file, `sofy-catalog.json`, which should be added to a new `sofy` folder in the Helm chart directory.

The SoFy onboarding pipeline automatically updates internal development and test SoFy environments (e.g., https://products.pnpsofy.com) with all changes from any of the GA, DEMO, BETA, or INCUBATOR folders of the [sofy-catalog-content](https://github01.hclpnp.com/kubernetes/sofy-catalog-content) GitHub repository. 

**Note**: 
- All of the [generic metadata](#generic-metadata) marked **[required]** must be included to onboard anything into the SoFy Catalog.
- `openAPISpecs` is required if any public REST APIs are available.
- The `.helmignore` file should include an entry for `sofy/`.
- Changes made exclusively within the `./sofy` directory do not require a Helm chart version change.

The SoFy onboarding pipeline reads metadata from `./sofy/sofy-catalog.json` and other files from within this file structure:
  
```
sofy-catalog-content
│   README.md
│   requirement.md
└───GA
│   └───my-helm-chart
│   │   └───sofy
│   │   │   └───docs
│   │   │   │   │   author-api.yaml
│   │   │   │   │   cart-api.yaml
│   │   │   │   │   documentation.md
│   │   │   └───images
│   │   │   │   │   icon.svg
│   │   │   └───public
│   │   │   │   │   public_cleared_sample_code.zip
│   │   │   │   │   public_demo_scripts.zip
│   │   │   │   │   video_link_thumbnail.png
│   │   │   └───tests
│   │   │   │   sofy-catalog.json
│   │   └───templates
│   │   │   .helmignore
│   │   │   Chart.yaml
│   │   │   values.yaml
│   └───informix
└───BETA
└───INCUBATOR
```

### Generic Metadata

Required for all SoFy catalog items, except as noted, SoFy generic metadata includes the product or service name, icon, description, documentation, and more. Generic metadata requires changes to both the `./Chart.yaml` and `./sofy/sofy-catalog.json` files (located within the solution’s Helm chart directory).

**`./Chart.yaml` generic metadata**

```yaml
apiVersion: v2
name: oneguru-data
description: OneGuru demo pack helm chart for K8s.
version: 1.1.1
appVersion: 1.16.0-Preview
```

| Metadata | Meaning | Example |
| -------- | ------- | ------- |
| **apiVersion** | [required] Must be set to `v2`. | `apiVersion: v2` |
| **appVersion** | [optional] The version number of the underlying HCL application (as opposed to the Helm chart version, which is specified in `version`).<br /><br />Contents designated as “preview” in the SoFy catalog are only visible to internal HCL users. The default is non-preview. To add a visual indicator in the catalog, append `-Preview` to the appVersion. | `appVersion: 1.16.0`<br /><br /> ...or...<br /><br /> `appVersion: 1.16.0-Preview` |
| **version** | [required] Product or application version | `version: 1.1.1` |


**`./sofy/sofy-catalog.json` generic metadata**

Note that in the example below, `documentation` is used to display a single user markdown document, displayed under the **User Guide** tab. Use a `docs` array of objects to specify multiple user documents (using `displayName` and `file`). Each document will be displayed under its own tab.

```json
{
    "name": "SoFy HTTP Debug",
    "description": "A simple microservice to debug HTTP requests. It shows information about the received request including the headers, body, cookies, etc.",
    "type": "product",
    "documentation": "./docs/documentation.md",
    "openAPISpecs": [
      {
        "displayName": "ACP Group Sub API",
        "description": "Restful services to a collection of policy group subscriptions.",
        "file": "./docs/api-acp-group-sub.yaml"
      },
      {
        "displayName": "Approval Status API",
        "description": "This API provides RESTful services to manage approval status resources.", 
        "file": "./docs/api-approval-status.json"
      },
      {
        "displayName": "Broadcast Job API",
        "description": "This class provides restful services to a collection of broadcast job statuses.", 
        "file": "./docs/api-broadcast-job.json"
      }
    ],
    "icon": "./images/icon.svg",
    "public": [
      "./public/public_cleared_sample_code.zip",
      "./public/public_demo_scripts.zip", 
      "./public/video_link_thumbnail.png"
    ],
    "tags": [
        "SoFy",
        "debugging",
        "http",
        "request",
        "transport",
        "snoop"
    ],
    "hclSWCategory": "SoFy",
    "status": "ga"
}
```

| Metadata | Meaning | Example |
| -------- | ------- | ------- |
| **description** | [required] A short description to display in SoFy catalog, under 170 characters. See [Description guideline](./docs/catalog-description-guidelines.md) | "A fast and scalable database server for managing traditional, object-relational, and dimensional data." |
| **documentation** | [required] relative path to your user guide (markdown doc) for your catalog entry's details page, displayed under "User Guide".<br /><br />See [markdown docs templates](./docs)<br /><br />**Note**: DO NOT use this value for Demo Docs entries with multiple markdown documents. Instead, use the `docs` array as described in the Demo Docs metadata section of this document | `"documentation": "./docs/documentation.md"` |
| **hclSWCategory** | [required] HCL Software pillars used for filtered view in SoFy catalog.  Must be one of: `Automation` / `Commerce & Marketing` / `Data Management` / `DevOps` / `Digital Solutions` / `Mainframes` / `SoFy`.<br /><br />See [HCL SW Products by Pillars on https://www.hcltechsw.com/](https://www.hcltechsw.com/wps/portal/products) | `"hclSWCategory": "Data Management"` |
| **icon** | [optional] A single .svg file to be used for displaying the product’s logo in the SoFy catalog. If missing, a generic logo will be substituted. The logo will be displayed at a size of 64x64 pixels. | `"icon": "./images/icon.svg"` |
| **incompatibilities** | [opotional] Use this property to prevent SoFy users from adding incompatible catalog contents in the same SoFy solution. If content A is incompatible with B, both need to specify this property. See [more details here](#incompatibilities) | `"incompatibilities": [{ "chartName": "hcl-wa-server-prod", "chartVersion": "*" }]` |
| **name** | [required] Display name of your tile entry in SoFy catalog. For [Demo Packs](#demo-packs), must end with " Demo Pack", for [Demo Docs](#demo-docs), must end with " Demo".  | "HCL OneDB", "OneDB Demo Pack", "Commerce Demo" |
| **openAPISpecs** | [optional] An array containing the name, description, and file location of the `.yaml` or `.json` specification documents for all public REST APIs (must be somewhere within the `sofy` folder tree). These will be displayed as tags in your product's details page, as well as listed in a [new HCL API Directory](./docs/images/HCL-API-Directory-on-SoFy.png) where all HCL API docs are available from a central location.<br /><br />The old `openAPISpec` metadata has been deprecated. This new `openAPISpecs` metadata must contain: 1) API display name (35 characters max); 2) API description (150 characters max); and 3) the relative path to the individual spec file. See  [API name and description guideline](./docs/API-Description-Guidelines.md) | `"openAPISpecs": [{"displayName": "ACP Group Sub API", "description": "Restful services to a collection of policy group subscriptions.", "file": "./docs/api-acp-group-sub.yaml"}]` |
| **public** | [optional] An array of public artifacts (e.g., images, file downloads, etc.) which need to be made avilable from a public URL for use in the markdown user documentation. The actual files should also be included in the `./sofy/public/` folder.<br /><br />Note that you are fully responsible for verifying everything pushed to this repository has been legally cleared by HCL for free download from the Internet.<br /><br />All files will be automatically uploaded to `https://hclcr.io/files/sofy/catalog/[chart-name]/generic/`. | `"public": ["./public/video_link_thumbnail.png", "./public/public_demo_scripts.zip", "./public/public_cleared_sample_code.zip"]` |
| **status** | [optional] Contents designated as “preview” in the SoFy catalog are only visible to internal HCL users. The default is non-preview. Specify `"status": "preview"` for preview contents.  | `"status": "preview"` |
| **tags** | [optional] An array of tags users can use to search for this entry in the SoFy Catalog. | `"tags": ["database", "persistence", "beta"]` |
| **type** | [required] Content type in SoFy catalog, must be one of: `product`, `service`, `demopack`, `demodoc`, or `proxy`.<br /><br />Use `demopack` for demo packs. `demodoc` and `proxy` are special type of entries in SoFy catalog which provide user documents only. They cannot be included in a SoFy solution. | `"type": "product"` |


### Demo Packs metadata

**Demo Packs** are a special type of content in the SoFy catalog which provide additional capabilities and/or content to standard SoFy products. For instance, available Demo Packs include a customized demo for HCL Commerce, and an integration between HCL Commerce and HCL Digital Experience.

Just like individual products or modules, Demo Packs have their own standalone Helm charts. As a part of these charts, the products needed by each Demo Pack (such as HCL Commerce or HCL BigFix) must also be included in the Demo Pack specification.

All [generic metadata](#generic-metadata) applies.

Requires additional changes to `./Chart.yaml`, `./sofy/sofy-catalog.json`, and `./values.yaml` as specified below.


**`./Chart.yaml` metadata changes for Demo Packs**, 

Must specify all product and/or module dependencies. That is, any products or modules required for the Demo Pack to work properly. For example, the *HCL Commerce & Digital Experience Integration Demo Pack* requires both HCL Commerce and HCL Digital Experience, so each of those products is listed as a dependency, as shown below.

```yaml
apiVersion: v2
name: oneguru-data
description: A Helm chart for Kubernetes
version: 1.1.1
appVersion: 1.16.0-Preview
dependencies:
- name: onedb-product
  # Use version range to support the latest product version. `^0.1.17` is equivalent to `>=0.1.17` and `<1.0.0`
  version: ^0.1.17
  repository: "https://github01.hclpnp.com/pages/kubernetes/helm-hub/"
  condition: global.outsideSoFySolution
```

| Metadata | Meaning | Example |
| -------- | ------- | ------- |
| **dependencies** | [required] An array of all products or services required by the Demo Pack. Each elements of the array must include each of the following name-value pairs. | |
| **condition** | [required] Use the value shown here.<br /><br />This value is defaulted to `false` in `values.yaml`, which means all dependencies will be installed as peers to the Demo Pack so they can be shared by all Demo Packs included in the SoFy Solution.<br /><br />**Note**: When installing the standalone demo chart outside of a SoFy solution, override this value to true will make sure the product chart gets installed as a child of the demo. | `condition: global.outsideSoFySolution`|
| **name** | [required] The HCL product’s Helm chart name. Must match exactly. | `name: domino` |
| **repository** | [required] For HCL products, always use the default value shown here. | `repository: https://github01.hclpnp.com/pages/kubernetes/helm-hub/` |
| **version** | [required] The HCL product’s Helm chart version number or range.<br /><br />For complete version-matching syntax, see [https://github.com/Masterminds/semver#checking-version-constraints](https://github.com/Masterminds/semver#checking-version-constraints) | `version: ^0.1.10`|


**`./sofy/sofy-catalog.json` metadata changes for Demo Packs**

```json
{
    "type": "demopack",
    "name": "HCL OneDB Demo Pack",
    "description": "A demo application for the SoFy HTTP Debug service. It shows information about the received request including the headers, body, cookies, etc."
}
```

| Metadata | Meaning | Example |
| -------- | ------- | ------- |
| **description** | [required] A short description for display on the Demo Pack tile in the SoFy catalog. Limited to 170 characters. See [Description guideline](https://github01.hclpnp.com/pages/kubernetes/sofy-catalog-content/docs/catalog-description-guidelines.html) | `"description": "A demo application for the SoFy HTTP Debug service. It shows information about the received request including the headers, body, cookies, etc."` |
| **name** | [required] Display name of the tile as it will appear in the SoFy catalog. For Demo Packs, this must begin with the product name and end with “Demo Pack”.  | `"name": "HCL OneDB Greenlight Demo Pack"` |
| **type** | [required] Must be `demopack` for demo packs. | `"type": "demopack"` |

**`./values.yaml` metadata changes for Demo Packs**

Only one change is required. Add (or modify) `global.outsideSoFySolution` with a value of `false`.

```yaml
global:
  outsideSoFySolution: false
```

### Demo Docs and Proxy metadata

**Demo Docs** are a special type of content in the SoFy catalog, intended to provide documentation for the built-in demos included in some of the products listed in the SoFy Catalog. Demo Docs entries do not have Helm charts, so cannot be directly included in SoFy solutions, but do have their own tiles in the SoFy UI. A link to the latest version of the parent product (e.g., HCL Domino, or HCL OneDB) is provided on the Demo Doc’s details page. 

Each parent product may only have one Demo Doc entry in SoFy catalog, but that entry can contain as many sets of documentation (in the form of markdown documents) as needed. 

**Proxy** entries are for products not yet cloud native-ready or otherwise unable to be included directly in SoFy. Proxy tiles are very similar to Demo Docs in that they have no associated Helm charts, but Proxy tiles do not include any links to their associated products in SoFy. Instead, they contain markdown documents which generally include hyperlinks to websites or online demos outside of SoFy.

All [generic metadata](#generic-metadata) applies.

Requires metadata adjustments to `./sofy/sofy-catalog.json` only.

**`./sofy/sofy-catalog.json` metadata changes for Demo Docs**:

```json
{
    "name": "HCL Commerce Personalization Demo",
    "type": "demodoc",
    "docs": [
      {
        "displayName": "Credit Card Onboard",
        "file": "./docs/credit-card-onboard.md"
      },
      {
        "displayName": "Credit Card Activation",
        "file": "./docs/credit-card-activation.md"
      }
    ],
    "requiredProducts": ["hcl-commerce", "hcl-accelerate"]
}
```

| Metadata | Meaning | Example |
| -------- | ------- | ------- |
| **docs** | [required] An array containing the display names and file paths for the markdown documents used to describe the built-in demos.<br /><br />Products on SoFy may only have one tile in the catalog for demo documentation, but that tile can include any number of markdown documents for demo scenarios.<br /><br />Each document will be displayed under its own tab on the tile’s details page. The display name for each demo is limited to 35 characters. | `"docs": [{"displayName": "Credit Card Onboard", "file": "./docs/credit-card-onboard.md"}]` |
| **name** | [required] Display name of the tile as it will appear in the SoFy catalog. For Demo Docs, must start with the product name and end with the word “Demo”.  | `"name": "HCL Commerce Personalization Demo"` |
| **requiredProducts** | [required for Demo Docs] An array containing the Helm chart names of all products required for the demos included in the documentation. Used to link back to each product’s latest version in the SoFy catalog. Not used for Proxy entries. | `"requiredProducts": ["hcl-commerce", "hcl-accelerate"]` |
| **type** | [required] Must be `demodoc` for Demo Docs, or `proxy` for Proxy entries. | `"type": "demodoc"` |

### Hybrid Mode metadata

By default, SoFy solutions are deployable to any Kubernetes environment, including the SoFy sandbox, by anyone with (a) access to SoFy, and (b) the necessary FlexNet entitlements. **Hybrid Mode** makes contents visible and deployable in the SoFy sandbox by all SoFy users (internal and external) while restricting the ability to download or deploy SoFy Solutions into Kubernetes clusters other than the SoFy sandbox to internal HCL users. Internal HCL users can see, deploy to the SoFy sandbox, and download to deploy solutions elsewhere. 

Must use the following metadata and values.  All [generic metadata](#generic-metadata) applies.


**`./Chart.yaml` metadata change for Hybrid Mode**

Append `-Preview` to your `appVersion` value:

```yaml
apiVersion: v2
name: oneguru-data
description: OneGuru demo pack helm chart for K8s.
version: 1.1.1
appVersion: 1.16.0-Preview
```

| Metadata | Meaning | Example |
| -------- | ------- | ------- |
| **appVersion** | [required] Append `-Preview` to the `appVersion` number for hybrid entries. | `appVersion: 1.16.0-Preview` |


**`./sofy/sofy-catalog.json` metadata changes for Hybrid Mode**

Change `status` to `ga`.

```json
{
    "status": "ga",
    "name": "SoFy HTTP Debug Demo Pack",
    "description": "A demo application for the SoFy HTTP Debug service. It shows information about the received request including the headers, body, cookies, etc."
}
```

| Metadata | Meaning | Example |
| -------- | ------- | --------- |
| **status** | [required] Use `ga` for **hybrid mode**. | `"status": "ga"` |


**`./values.yaml` metadata change for Hybrid Mode**

In the `global` settings, use `global.hclPreviewImageRegistry` with a value of `hclcr.io/sofy-hcl-users` (instead of `global.hclImageRegistry`, with a value of `hclcr.io/sofy`).

This is a point about which there is often some confusion. Hybrid entries must use `global.hclPreviewImageRegistry` rather than `global.hclImageRegistry` as the location for all application images. This is done to ensure they remain accessible only by internal HCL users. However, SoFy content creators are often confused by the value `hclcr.io/sofy-hcl-users` since they know their content is stored in the GCR registry (`gcr.io/blackjack-209019`). This is the registry used during Helm chart development for testing, as well as for all SoFy sandbox deployments. However, during the process of promoting products to production, the GCR images will be automatically promoted to the Harbor Preview registry (`hclcr.io/sofy-hcl-users`) by the SoFy team. 


```yaml
global:
  hclPreviewImageRegistry: hclcr.io/sofy-hcl-users
  hclImagePullSecret: ''
```
| Metadata | Meaning | Example |
| -------- | ------- | --------- |
| **global.hclPreviewImageRegistry** | [required] Docker images for preview catalog items are stored separately from those of released items. If the preview item includes a Docker image, this setting is required and should be set to `hclcr.io/sofy-hcl-users`. | `global: hclPreviewImageRegistry: hclcr.io/sofy-hcl-users` |

### Incompatibilities metadata

Incompatibilities are conflicts between SoFy catalog entries which prevent them from being included in the same SoFy solution. For example, two demo packs may each require different settings for the same value. SoFy `incompatibilities` values are used to define these potential points of overlap. The SoFy UI will prevent conflicting contents—such as two incompatible Demo Packs—from being included in the same SoFy Solution.

When adding new content to the SoFy catalog, it’s important to understand all existing content which might conflict with the new content and to document those incompatibilities, as described below, *in both entries*. 

#### Defining Incompatibilities

If A and B cannot be used in the same solution, the property `incompatibilities` must be included in the `./sofy/sofy-catalog.json` file of *both* A and B. `incompatibilities` is an array of objects containing the `chartName` and `chartVersion` for each conflicting product. In other words, the `incompatibilities` array for product A would include product B's chart name and version, while product B's `incompatibilities` array would include product A's chart name and version. Each array element represents the other end of the incompatibility pair, using the Helm chart name and semver range (or "\*" for all versions).

For example, if `hcl-workload-automation-prod` is incompatible with all versions of `hcl-wa-server-prod`, `hcl-wa-console-prod`, and `hcl-wa-dyn-agent-prod`, it's `./sofy/sofy-catalog.json` file would include the following:

```json
{
  "incompatibilities": [
    { 
      "chartName": "hcl-wa-server-prod",
      "chartVersion": "*"
    }, 
    { 
      "chartName": "hcl-wa-console-prod",
      "chartVersion": "*"
    }, 
    { 
      "chartName": "hcl-wa-dyn-agent-prod",
      "chartVersion": "*"
    }
  ]
}
```

Then, each of those conflicting products' Helm charts would need to include `hcl-workload-automation-prod` in their `incompatibilities` arrays. 

From `hcl-wa-server-prod/sofy/sofy-catalog.json`:

```json
{
  "incompatibilities": [
    { 
      "chartName": "hcl-workload-automation-prod",
      "chartVersion": "*"
    }
  ]
}
```

From `hcl-wa-console-prod/sofy/sofy-catalog.json`:

```json
{
  "incompatibilities": [
    { 
      "chartName": "hcl-workload-automation-prod",
      "chartVersion": "*"
    }
  ]
}
```

And from `hcl-wa-dyn-agent-prod/sofy/sofy-catalog.json`:

```json
{
  "incompatibilities": [
    { 
      "chartName": "hcl-workload-automation-prod",
      "chartVersion": "*"
    }
  ]
}
```

## Helm Chart Requirements

### Chart Acceptance

#### 1. Helm 3 Chart
  - Must be a Helm 3 chart.
  - Must pass the Helm linter test. Use `helm lint` to verify.
  - Must specify `/sofy/` in `.helmignore` file to exclude SoFy metadata, public files, etc. from being included in the chart tarball
#### 2. Helm Install
  - The Chart must `helm install` successfully
    - To be installable by users in the SoFy sandbox, the chart must be sufficietly self-contained that the application is installable with default configurations.
  - Must install successfully into Kubernetes 1.17.
    - K8S 1.16 is the supported version for Aug. 2020 release. Note [API deprecations in K8S 1.16](https://kubernetes.io/blog/2019/07/18/api-deprecations-in-1-16/)
#### 3. Self Contained
  - All dependencies and any content required for default configuration usage must be included so that it is **installable out-of-the-box in SoFy Sandbox**.
#### 4. Alias or Name Override for Dependency Charts
  - To avoid collision of the same dependencies (e.g. MongoDB, Redis, OneDB or Informix) when multiple products are included in the same SoFy Solution.
#### 5. Self Healing
  - Anytime Kubernetes restart the pods, the application should continue to function without any loss of data.
#### 6. Upgrade
  - `helm upgrade` and `helm rollback` should work with supported upgradable versions of the Helm chart
    - The new chart version may include new Docker images, new image versions, chart changes, or any combination of these.
#### 7. Scaling
  - Allow horizontal scaling, if applicable to your product.

*Example:*  
- All Helm charts in the [sofy-catalog-content](https://github01.hclpnp.com/kubernetes/sofy-catalog-content/tree/master/GA) GitHub repository for GA are good examples of these requirements in use.

### SoFy Integration

When users browse the SoFy catalog for HCL software, they can easily build a custom *SoFy Solution* consisting of one or more software products from the catalog.  

In order for any HCL catalog entries to be installable and accessible from a *SoFy Solution*, the Helm chart needs to enable:

* SoFy Integration
  - [1. FlexNet License](#1-flexnet-license)
  - [2. SoFy Solution Routing](#2-sofy-solution-routing)
  - [3. Global Configs](#3-global-configs)
  - [4. Solution Console UI Integration](#4-solution-console-ui-integration)
  - [5. Pods Requirements](#5-pods-requirements)
  - [6. Getting Sofy Domain in a Container](#6-getting-sofy-domain-in-a-container)
  - [7. Kubernetes Resource Naming Best Practice](#7-kubernetes-resource-naming-best-practice)

Teams that wish to use the same Helm chart for release channels outside of SoFy can hide the SoFy-required elements conditionally based on `.Values.global.sofySolutionContext` which is only set to `true` in SoFy solution charts.

#### 1. FlexNet License
 
##### Requirements

1. FlexNet runtime license check is required by your product container(s) for GA. This is used as an anti-piracy mechanism.  

2. Customer's license information can be set during the SoFy Solution Helm chart install using either of the following approaches:

When customer has a single license for all products included in the solution:
```bash
helm install my-release my-solution-chart.tgz --set hclFlexnetURL=[my-flexnet-server-url] --set hclFlexnetID=[my-flexnet-id]`
```

When the customer has multiple licenses included in the solution, individual licenses can be designated toward specific products. This example sets a second license for the `onedb` child chart in the solution:
```bash
helm install my-release my-solution-chart.tgz \
--set hclFlexnetURL=[my-flexnet-server-url1],hclFlexnetID=[my-flexnet-id1] \
--set onedb.hclFlexnetURL=[my-flexnet-server-url2],onedb.hclFlexnetID=[my-flexnet-id2]
```

##### How to Enable

To enable this in your product container(s), they should be able to accept FlexNet license configurations at startup time, usually via environment variables.

Enabling this in your Helm chart requires adjustments in three places:

1) Expose the following common variables in `values.yaml` as empty strings.

*values.yaml Example:*

```yaml
# variables for FlexNet license
hclFlexnetURL: ''
hclFlexnetID: ''
```

2) To enable a multiple-license scenario, create named template(s) such as the following in `./templates/_helpers.tpl`, or your own .tpl file, using values from either local or global variables. Note that if variable names conflict, local variables take precedence.

*_helpers.tpl:*

```yaml
{{/*
FlexNet License URL
*/}}
{{- define "license.url" -}}

{{- if .Values.hclFlexnetURL -}}
{{ tpl .Values.hclFlexnetURL . }}
{{- else if .Values.global.hclFlexnetURL -}}
{{ tpl .Values.global.hclFlexnetURL . }}
{{- end -}}
{{- end -}}

{{/*
FlexNet License ID
*/}}
{{- define "license.id" -}}

{{- if .Values.hclFlexnetID -}}
{{ tpl .Values.hclFlexnetID . }}
{{- else if .Values.global.hclFlexnetID -}}
{{ tpl .Values.global.hclFlexnetID . }}
{{- end -}}
{{- end -}}
```

3) Use the named templates to pass the FlexNet license to your containers.

*deployment.yaml example:*

```yaml
      containers:
        - image: "{{ .Values.global.hclImageRegistry }}/{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          env:
            - name: "HCL_LICENSE_SERVER_URL"
              value: "{{ include "license.url" . }}"
            - name: "HCL_LICENSE_SERVER_ID"
              value: "{{ include "license.id" . }}"
```

#### 2. SoFy Solution Routing

SoFy uses [Ambassador](https://www.getambassador.io/) to provide common routing for all products and services included in a SoFy solution.  

When Sofy Solutions are deployed in customers own Kubernetes cluster, all the services are exposed via Ambassador. Application users access the service endpoints via Ambassador IP address.  

When the solutions are deployed in a Sofy Sandbox, there is an extra layer of LoadBalancer deployed on top of Ambassador. Along with the Ambassador deployed for each solution, there is an Nginx LoadBalancer in the Sandbox to expose the Ambassador endpoints.

![Routing](./docs/images/Routing.png?raw=true "Routing in Sofy Solution")

##### Default Routing

By default SoFy solutions automatically create public routes for each Kubernetes service in the Helm chart. 

*Example:*

The [SoFy Debug Service Helm chart](https://github01.hclpnp.com/kubernetes/sofy-catalog-content/tree/master/GA/snoop) does not define any routing (i.e., ingress from the Helm template is disabled). However, when that service is included in a SoFy Solution, it is automatically assigned a public route at: `http://snoop.{solution-ip}.nip.io`.

*Note*: 

If your service is internal to the cluster only, you can disable the default public route for that service in your chart, or in any child charts. See [sofy-config.yaml](#4-solution-console-ui-integration).

##### Custom Routing and Exposing TCP Ports

When products have more advanced routing requirements, those can be configured through [Ambassador Mapping CRD](https://www.getambassador.io/docs/latest/topics/using/intro-mappings/).

The value required by the `ambassador_id` property of the Mapping CRD of any SoFy solution is available from a global variable in values.yaml: `{{ .Values.global.ambassadorID }}`.

*Example:*

The Workload Automation server chart defines a [custom routing with ambassador mapping CRD](https://github01.hclpnp.com/kubernetes/sofy-catalog-content/blob/master/BETA/hcl-wa-server-prod/templates/ambassador-mapping.yaml).


TCP ports can be exposed in a similar way through either a [TCPMapping](https://www.getambassador.io/docs/latest/topics/using/tcpmappings/) or [annotaion](https://kubernetes.io/docs/concepts/overview/working-with-objects/annotations/). 

Exposing TCP ports will also require the ports be reserved on SoFy's ambassador, please contact a SoFy dev so the desired ports can be reserved. Due to a [known limitation with Ambassador](https://github.com/datawire/ambassador/issues/1965), TCP ports cannot be exposed through the SoFy Sandbox environemnt. This limitation should be documented in your service markdown under the Access URL's section. See the [OneDB SQL](https://github01.hclpnp.com/Kunj-Patel/sofy-catalog-content/blob/master/GA/onedb-sql/sofy/docs/documentation.md) markdown for a good example on documenting exposed TCP ports. 

*Examples:*

The HCL Launch chart exposes a TCP endpoint through a [TCPMapping yaml file](https://github01.hclpnp.com/Kunj-Patel/sofy-catalog-content/blob/master/GA/hcl-launch-server/templates/ambassador-mapping.yaml).

The OneDB SQL chart exposes a TCP endpoint through annotations. See [Values.yaml](https://github01.hclpnp.com/Kunj-Patel/sofy-catalog-content/blob/master/GA/onedb-sql/values.yaml) & [service.yaml](https://github01.hclpnp.com/Kunj-Patel/sofy-catalog-content/blob/master/GA/onedb-sql/templates/service-sqli.yaml)

Note: To easily toggle exposing TCP endpoints, the annotation or TCPMapping should be wraped with a IF statment using the disableAccessControl variable. See the [OneDB SQL](https://github01.hclpnp.com/Kunj-Patel/sofy-catalog-content/blob/master/GA/onedb-sql/values.yaml) annotation for an example. 


*Custom routing in SoFy Charts*

Imagine a product wants to have a custom domain defined for a Kubernetes service. To define such an end point in a SoFy Helm chart, create a dummy service with the custom domain as its service name in the Helm chart's `templates` folder.

Dummy Kubernetes service example: 

```YAML
{{- if .Values.global.sofySolutionContext }}
---
apiVersion: v1
kind: Service
metadata:
  name: {{ .Release.Name }}-custom-domain
spec:
  type: ClusterIP
  ports:
    - port: 80
---
{{- end}}
```

Then create an Ambassador mapping for the same to point to the actual Kubernetes service.

Ambassador Mapping Example: 

```YAML
apiVersion: getambassador.io/v1
kind: Mapping
metadata:
  name: crd-custom-domain
spec:  
  prefix: /.*
  host: custom-domain.{{ .Values.global.domain }}
  ambassador_id: {{ .Values.global.ambassadorID }}
  case_sensitive: false
  host_regex: true
  prefix_regex: true
  rewrite: ""
  timeout_ms: 120000
  connect_timeout_ms: 1200000
  idle_timeout_ms: 1200000
  bypass_auth: {{ .Values.disableAccessControl }}
  service: http://actualservice:8081
```

Note: The `host` value of the Ambassador mapping, and the dummy Kubernetes service name (the part that comes after `.Release.Name`) must be same. In this example it is `custom-domain`.

Once the Ambassador mapping has been defined, update `sofy-config.yaml` to include the dummy Kubernetes service so that its endpoints will be shown in the SoFy Solution Console UI.

Example : sofy-config.yaml
```JSON
  "{{ .Release.Name }}-custom-domain":{
          "disablePublicRoute" : "true",
          "disableAccessControl" : "true",
          "contextRoots": {
            "ui": {
              "Custom Page ": "/newPage.html"
            },
            "health": {
              "checktype": "http",
              "endpoint": "/"
            }
          }
        }
```
Note: Do not add `additionalLogs` to dummy service name as there will be no pods/containers attached to it. 

To get additional logs, define the logs for the actual Kubernetes service which points to the pods and containers.

Example : sofy-config.yaml
```JSON
 "actualservice": {
          "disablePublicRoute" : "true",
          "disableAccessControl" : "true",
          "additionalLogFiles" : {
            "actualservice": {
              "Products logs": "/logsFolder/"
            }
          }
        }
```
#### 3. Global Configs

##### Chart.yaml file

Chart.yaml provides metadata about the helm chart. To onboard SoFy catalog, it must include the following:

| Properties        | Value |
| ----------------- | ------ |
| apiVersion        | [required] v2 for Helm 3 charts |
| appVersion        | [required] Product version included in this chart. If the chart is targeted for Beta in SoFy, append with "-Beta". For example "11.1.0.2", "14.10.FC3-Beta", or "9.2.0.19-pvu" etc. Semver is not required.|
| name              | [required] Name of the chart, must be unique in SoFy catalog, thus in the [sofy-catalog-content](https://github01.hclpnp.com/kubernetes/sofy-catalog-content) repository. |
| version           | [required] Version of the chart, must be updated each time any chart change is pushed to the [sofy-catalog-content](https://github01.hclpnp.com/kubernetes/sofy-catalog-content) repository to guarantee the new version is onboarded. Must be [semver](https://semver.org/) with *Major.Minor.Patch* only. |  


*Example:*

```yaml
apiVersion: v2
name: informix
version: 0.11.0
appVersion: 14.10.FC3-Beta
description: HCL Informix Helm chart for Kubernetes
```

##### values.yaml file

Global and local variables enable common configurations to be overwritten at the solution level as well as at each individual product level. For example, users can set their FlexNet license once for the entire solution, rather than having to set it individually for each product or service included in the solution. Of course, different FlexNet licenses can be set for one or more products if needed.

| Configuration | Value |
| ------------- | ----- |
| global.hclImageRegistry | [required] Global registry variable for **GA images**. Use `hclcr.io/sofy`.<br /><br />**Note**: All SoFy solutions require a value be provided for at least one of `global.hclImageRegistry` or `global.hclPreviewImageRegistry`. Both are not needed for most SoFy Solutions, although some Solutions may require them. |
| global.hclImagePullSecret | [required] Global variable with “” empty value by default. A secret with user’s own registry credentials is required at helm chart install time for non-sandbox installs.<br /><br />For example: <br />`helm install my-release my-chart.tgz --set global.hclImagePullSecret=[my-registry-pull-secret]` |
| global.hclPreviewImageRegistry | [required] Global registry variable for **Preview images**. Use hclcr.io/sofy-hcl-users.<br /><br />**Note**: All SoFy solutions require a value be provided for at least one of `global.hclImageRegistry` or `global.hclPreviewImageRegistry`. Both are not needed for most SoFy Solutions, although some Solutions may require them. |
| hclFlexnetID | [required] Empty by default (‘’). User’s own FlexNet ID can be passed in at install time. The value must be properly set to the containers that need it for license check. <br /><br />For example: <br />`helm install my-release my-chart.tgz --set hclFlexnetURL=[my-flexnet-server-url] --set hclFlexnetID=[my-flexnet-id] --set informix.hclFlexnetURL=[my-informix-flexnet-server-url] --set informix.hclFlexnetID=[my-informix-flexnet-id]` |
| hclFlexnetURL | [required] Empty by default (‘’). User’s own FlexNet Server URL can be passed in at install time. The value must be properly set to the containers that need it for license check. <br /><br />For example: <br />`helm install my-release my-chart.tgz --set hclFlexnetURL=[my-flexnet-server-url] --set hclFlexnetID=[my-flexnet-id]` |

Since the Docker image registry is set by `global.hclImageRegistry`, it's necessary to have a separate variable for your image path within that registry. When image paths are consistent across different registries, having separate variables allows easy switching between test and production registries, by simply overwriting the `global.hclImageRegistry` value at install time.

*values.yaml Example:*

```yaml
# global variables for docker registry access
global:
  hclImageRegistry: hclcr.io/sofy  # Default production registry. Push your images to "gcr.io/blackjack-209019".
  hclImagePullSecret: ''
# local variables for FlexNet cliense  
hclFlexnetURL: ''
hclFlexnetID: ''
# docker image path and tag inside the image registry
image:
  repository: services/test-data-synth/test-data-synth
  tag: 1.5.0
  pullPolicy: IfNotPresent
```

With separate image registry, image path, and image tag variables in `values.yaml`, as shown above, all pods' templates must specify the full image pull path in `pod.spec.containers.image` by concatenating them.

Use the template `{{ tpl .Values.hclFlexnetURL . }}` and `{{ tpl .Values.hclFlexnetID . }}` to pass the values as needed. The example below illustrates passing these values to a pod by using environment variables:

*deployment.yaml Example:*

```yaml
apiVersion: apps/v1
kind: Deployment
spec:
  template:
    spec:
      {{- if .Values.global.hclImagePullSecret }}
      imagePullSecrets:
      - name: {{ .Values.global.hclImagePullSecret }}
      {{- end }}
      containers:
        - image: "{{ .Values.global.hclImageRegistry }}/{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          imagePullPolicy: {{ .Values.image.pullPolicy }}
          name: {{ .Chart.Name }} ## your own container name
          env:
            - name: HCL_LICENSE_SERVER_URL
              value: {{ include "license.id" . | quote }}
            - name: HCL_LICENSE_SERVER_ID
              value: {{ include "license.id" . | quote }}
```

**Note**: See [Flexnet license requirement](#1-flexnet-license) for For more information and examples on how to use named templates.

##### Useful global values

There can be times when global contexts are needed. Here is a collection of them and how they can be used:

- `global.sofySandboxContext` from values.yaml. This variable is not created in the generated solution chart. However when the solution is installed in SoFy Sandbox, it is created and will be set to `true`. This can be useful to know if the current install is in the Sandbox or not.  For example, it can be checked with:
  
```yaml  
{{- if .Values.global.sofySandboxContext }}
```

#### 4. Solution Console UI Integration

Each SoFy Solution includes an administrative front-end UI called *Solution Console*, designed to help users to easily navigate and troubleshoot all of the products and services included in the solution. Product-specific contents can be added to the Solution Console by editing the Kubernetes ConfigMap `./templates/sofy-config.yaml` which is included in the chart generated by SoFy. This ConfigMap contains a data property named `sofy-config.json` with a string value in JSON format.

`sofy-config.yaml` will continue to evolve, but currently you can use it to:

- [Provide links to Your End Points](#providing-links-to-your-end-points)
- [Disable default public routes and default authentication](#disable-default-public-routes-and-default-authentication)
- [Expose additional application log files](#expose-additional-application-log-files)
- [Specify required FlexNet feature names](#specify-required-flexnet-feature-names)

Each of these otions is described in more detail below.

**Note**: 
1. `sofy-config.yaml` is only required for SoFy integration and does not have any effect in your product chart stand-alone. It can even be safely removed for the chart stand-alone.
2. This is because the full content of `sofy-config.yaml` is enclosed in `{{- if .Values.global.sofySolution }}` and `{{- end }}`, and `global.sofySolution=true` only during solution creation time.
3. At sofy solution creation time, all `sofy-config.yaml` will be aggregated into one `sofy-config.yaml` for the solution only.

*Example sofy-config.yaml file:*

```yaml
{{- if .Values.global.sofySolution }}
apiVersion: v1
kind: ConfigMap
data:
  sofy-config.json: 
    {
        "chartName": "{{ .Chart.Name }}",
        "version": "{{ .Chart.Version }}",
        "displayName": "{{ .Values.displayName }}",
        "type": "demopack",
        "flexnetFeatures": [ "ODB_CPU" ],
        "links":[
          {
              "displayName": "Public Documentation",
              "type": "ui",
              "static": true,
              "url": "https://hclsw.com/doc"
          },
          {
              "displayName": "OneGuru Home Page",
              "type": "ui",
              "static": false,
              "subdomain": "oneguru",
              "path": "/home",
              "credentials": [
                  {
                      "displayName": "Default Admin Login",
                      "username": "admin",
                      "password": "admin"
                  },
                  {
                      "displayName": "Default User Login",
                      "username": "bob",
                      "password": "pass"
                  }
              ]
          },
          {
              "displayName": "OneDB API Root",
              "type": "apiroot",
              "static": false,
              "subdomain": "oneguru",
              "path": "/datasynth/1.0"
          },
          {
              "displayName": "OneDB REST API",
              "type": "openapiui",
              "static": false,
              "subdomain": "onedb-rest",
              "path": "/",
              "credentials": [
                  {
                      "username": "onedbsa",
                      "password": "onedb4ever"
                  }
              ]
          },
          {
              //expected connection URL: "jdbc:onedb://x.x.x.x.nip.io:3033"
              "displayName": "SQL Native OneDB Connection URL",
              "type": "nonhttp",
              "static": false,
              "prefix": "jdbc:onedb://",
              "port": "3033",
              "credentials": []
          },
          {
              //expected connection URL: "jdbc:db2://x.x.x.x.nip.io:3033"
              "displayName": "SQL Native OneDB Connection URL",
              "type": "nonhttp",
              "static": false,
              "prefix": "jdbc:db2://",
              "port": "3034",
              "credentials": []
          },
          {
              //expected connection URL: "x.x.x.x:8088", x.x.x.x being the external IP address of service "{{ .Release.Name }}-nrpc-service"
              "displayName": "Domino NRPC Connection",
              "type": "loadbalancer",
              "static": false,
              "serviceName": "{{ .Release.Name }}-nrpc-service",
              "prefix": "",
              "port": "8088",
              "credentials": []
          }
      ]
    }
{{- end }}
```

##### Providing Links to Your End Points 

The *SoFy Solution Console UI* can display links to your application’s public endpoints. This greatly improves the user experience by providing easy navigation to key points in your application from a common landing page. You can expose both static and dynamic links by declaring them in the `sofy-config.yaml` ConfigMap:

```yaml
data:
  sofy-config.json: 
     {
        "chartName": "{{ .Chart.Name }}",
        "version": "{{ .Chart.Version }}",
        "displayName": "{{ .Values.displayName }}",
        "type": "demopack",
        "flexnetFeatures": [ "ODB_CPU" ],
        "links":[
          {
              "displayName": "Public Documentation",
              "type": "ui",
              "static": true,
              "url": "https://hclsw.com/doc"
          },
          {
              "displayName": "OneGuru Home Page",
              "type": "ui",
              "static": false,
              "subdomain": "oneguru",
              "path": "/home",
              "credentials": [
                  {
                      "displayName": "Default Admin Login",
                      "username": "admin",
                      "password": "admin"
                  },
                  {
                      "displayName": "Default User Login",
                      "username": "bob",
                      "password": "pass"
                  },
                  . . .
              ]
           }
        ]
     }
```

###### Links Schema

**sofy-console UI** will support the following types:

| **links.type** | purpose | displayed as hyperlink in UI |
| ------------ | ------- | ---------------------------- |
| **apiroot** | URL to your REST API root for information purpose. | no |
| **loadbalancer** | None TCP protocals can't be routed through ambassador. like UDP or NRPC. Such `loadbalancer` type of k8s services have to be exposed directly via a load balancer from the cloud provider with an external IP and address and port. | no |
| **nonhttp** | TCP or other none HTTP protocol connection string. These are not accessible from SoFy sandbox install. Only accessible from non-sandbox installs. | no |
| **openapiui** | a link to a REST swagger UI or other API UI like GRAPHQL | yes |
| **ui** | a link to a web user interface. | yes |

A link can be either static or non-static link.  Non-static links are public endpoints routed by Ambassador.

| **links.static** | purpose | displayed as hyperlink in UI |
| ------------ | ------- | ---------------------------- |
| **true** | To a static link. Provide the full URL string in `url` | yes |
| **false** | Links relative to ambassador public domain or IP. | depends on `links.type` |


###### Links Schema (Deprecating soon)

**NOTE**: This schema will be deprecating after SoFy Solution Console UI added support for the new schema.

For HTTP end points, use one or all of the following under `contextRoots`:

| **contextRoots** | Purpose | Displayed as hyperlink in UI |
| ------------ | ------- | ---------------------------- |
| **apiroot** | The root URI of your APIs. Can be `/` | no |
| **openapiui** | The URI for swagger ui or other API UI | yes |
| **ui.key.DefaultLogin** | Default Admin credential as documented in user doc. Required if exist. | no |
| **ui.key.endpoint** | GUI URIs | yes |

For each of the above elements, include KEY-VALUE pairs:
- KEY: name of the link displayed in Solution Console
- VALUE: a relative URI path to your service's public route. UI will display the full URL by appending this value to the service's public route.

For Non-HTTP end points like TCP, use `others` under `contextRoots`. It can contain one or all of the following:

| **contextRoots** | Purpose | Displayed as hyperlink in UI |
| ------------ | ------- | ---------------------------|
| **others.key.connection** | TCP Endpoint URI | no |
| **others.key.DefaultLogin** | Default Admin credential as documented in user doc. Required if exist. | no |

- KEY: name of the link displayed in Solution Console

**Note**:
1. Surround links elements with `{{- if not .Values.disablePublicRoute }}` and `{{- end }}`.  When no public routes exist for the service, there should not be any public URLs available for end users. The service will still be available as an internal service inside the solution in this case.
2. See more details on `disablePublicRoute` below.

*Example:*

```yaml
{{- if .Values.global.sofySolution }}
apiVersion: v1
kind: ConfigMap
data:
  sofy-config.json: |-
    {
        "name": "{{ .Chart.Name }}",
        "version": "{{ .Chart.Version }}",
        "displayName": "{{ .Values.displayName }}",
        "flexnetFeatures": [ "ODB_CPU" ],
        "services": {
            "{{ include "test-data-synth.fullname" . }}": {
                {{- if not .Values.disablePublicRoute }}
                "contextRoots": {
                    "apiroot": {
                        "REST": "/datasynth/1.0",
                        "GRAPHQL": "/datasynth/gql"
                    },
                    "openapiui": {
                        "REST": "/openapi/ui",
                        "GRAPHQL": "/openapi/ui"
                    },
                    "ui": {
                        "Home Page": {
                          "endpoint": "/home",
                          "DefaultLogin": {
                            "username": "admin",
                            "password": "admin"
                          }
                        },
                        "Admin Console": {
                          "endpoint": "/console",
                          "DefaultLogin": {
                            "username": "consoleAdmin",
                            "password": "admin"
                        }
                    },
                    "others": {
                      "drda": {
                        "connection": "{{ .Values.global.domain }}:{{ .Values.service.externalPort }}",
                        "DefaultLogin": {
                          "username": "consoleAdmin",
                          "password": "admin"
                        }
                      }
                    }
                },
                {{- end }}
                "disablePublicRoute": "{{ .Values.disablePublicRoute }}",
                "disableAccessControl": "{{ .Values.disableAccessControl }}"
            }
        }
    }
{{- end }}
```

##### Disable default public routes and default authentication

Public routes are automatically created in SoFy solutions for all Kubernetes services exposed in your app’s Helm chart. However, in some use cases it may make sense to not expose certain services publicly, or to disable all public routes for your chart’s dependencies. For example, generally applications do not wish to publicly expose their database backend.

When Access Control Service (ACS) is selected during the creation of a SoFy Solution, by default ACS is also enabled for all included apps’ public routes in addition to protecting SoFy’s common routes such as the Solution Console UI. Applications which include authentication that has not been integrated with SoFy’s ACS may want to disable ACS for the services’ routes.

Adding `disablePublicRoute` and `disableAccessControl` elements to your Kubernetes services in `sofy-config.yaml` and in the chart’s `values.yaml` file will achieve this and make it possible to define a default behavior for the application, while still allowing a parent chart to overwrite this behavior as a dependency.

Note: When `disablePublicRoute` is set to `true`, `disableAccessControl` is effectively `true`, as well, because that element is not accessible.


*Example: sofy-config.yaml*

```yaml
{{- if .Values.global.sofySolution }}
apiVersion: v1
kind: ConfigMap
data:
  sofy-config.json: |-
      {
        "name": "{{ .Chart.Name }}",
        "description": "{{ .Chart.Description }}",
        "version": "{{ .Chart.Version }}",
        "displayName": "SoFy HTTP Debug",
        "services": {
          "{{ include "snoop.fullname" . }}-rest": {
            "disablePublicRoute" : "{{ .Values.disablePublicRoute }}",
            "disableAccessControl" : "{{ .Values.disableAccessControl }}"
          }
```

*Use Case 1: From the Informix chart's **values.yaml**, allow public route access,but disable ACS for Informix itself.*

```yaml
disablePublicRoute: false
disableAccessControl: true
```

*Use Case 2: From the Workload Automation chart's **values.yaml**, disable ACS for Workload Automation, and also disable both the public route access and ACS for Informix (as a child chart of Workload Automation).*

```yaml
disablePublicRoute: false
disableAccessControl: true

informix:
  disablePublicRoute: true
  disableAccessControl: true
```

##### Expose additional application log files

SoFy Solution users can access container console logs by default from Solution Console UI.  

If there are additional log files from within application containers that are useful for support, you can define them in the `sofy-config.json` section of `sofy-config.yaml` so they can also be accessed and downloaded from the SoFy Solution Console.

In the `sofy-config.json` data, add an `additionalLogFiles` section under the Kubernetes `services` of the specific product’s container, as shown in the example below.

**Note**: Each `additionalLogFiles` entry must be specified using the fully-qualified path to either (a) a specific log file, or (b) a folder of log files.

*Example*

```yaml
{{- if .Values.global.sofySolution }}
apiVersion: v1
kind: ConfigMap
data:
  sofy-config.json: |-
    {
        "name": "{{ .Chart.Name }}",
        "description": "{{ .Chart.Description }}",
        "version": "{{ .Chart.Version }}",
        "services": {
            "{{ include "wa.fullName" . }}": {
                "additionalLogFiles": {
                  "{{ .Chart.Name }}": {
                    "stdlist_jm": "/home/wauser/wadata/stdlist/JM/JobManager_message.log",
                    "stdlist_jmGw": "/home/wauser/wadata/stdlist/JM/JobManagerGW_message.log",
                    "engineServer": "/home/wauser/wadata/stdlist/appserver/engineServer/logs"
                  }
                },
                {{- if not .Values.disablePublicRoute }}
                "contextRoots": {
                    "ui": {
                        "Home Page": "/datasynth/appgui",
                        "Admin Console": "/datasynth/admingui"
                    }
                }
                {{- end }}
```

##### Specify required FlexNet feature names

Licensing issues are often hard to troubleshoot, both by individual users and by support. Details of important information such as FlexNet license features required by the product, users’ required entitlements, and so on, can often be hard to identify. Because of this, SoFy includes an ability to track and display FlexNet licensing requirements in the SoFy Solution Console.

For products which include FlexNet runtime license checks, add a `flexnetFeatures` array of required entitlement names to the `sofy-config.json` data of the `sofy-config.yaml` file:

`"flexnetFeatures": [ "HWA_Jobs","HID_CPU" ]`

*Example*

```yaml
{{- if .Values.global.sofySolution }}
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ include "wa.fullName" . }}-sofy-config
  labels:
    app.kubernetes.io/name: {{ include "wa.appName" . }}
    app.kubernetes.io/instance: {{ .Release.Name }}
data:
  sofy-config.json: |-
    {
        "name": "{{ .Chart.Name }}",
        "description": "{{ .Chart.Description }}",
        "version": "{{ .Chart.Version }}",
        "displayName": "HCL WorkLoad Automation",
        "flexnetFeatures": [ "HWA_Jobs","HID_CPU" ],
        "services": {
```

#### 5. Pods Requirements

##### Resource Requests and Limits

Kubernetes manages the allocation of resources to individual containers as described in [Resource requests and limits](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/). 

Helm charts in the SoFy catalog should specify at least the minimal resource requirements and, ideally, also a resource limit for each pod. Such configuration details can be included as a part of `values.yaml` so there’s a recommended default value, but advanced users can overwrite it at install time, if needed.

*Example*

In values.yaml file:

```yaml
resources:
  requests:
    cpu: 100m
    memory: 200Mi
  limits:
      cpu: 1
      memory: 1G
```

Then, in the `deployment.yaml` or `statefulset.yaml` file for `pod.spec.containers.resources`, can populate its value from `values.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
spec:
  template:
    spec:
      apiVersion: apps/v1
kind: Deployment
spec:
  template:
    spec:
      containers:
        - name: {{ .Chart.Name }}
          resources:
            {{- toYaml .Values.resources | nindent 12 }}
```

##### Health Checks

Kubernetes uses [Readiness, Liveness probes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/) to know whether pods are (a) ready for requests (Readiness probes), and (b) still alive or need a restart (Liveness probes). It’s important to provide these two health check probes for all pods. They can be implemented via command, HTTP GET request, or TCP socket. See [Kubernetes documentation](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/) for details.

**Note**: Readiness probes, especially when used for applications, can take some time to start up.

##### Labels and Selectors

[Kubernetes labels and selectors](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/) are key-value pairs used to identify and define attributes of Kubernetes resources. SoFy’s Solution Console uses these key-value pairs to identify and display all Kubernetes resources from your application.

Minimally, your pods and services **must** contain both the `app.kubernetes.io/name` and `app.kubernetes.io/instance` labels.

`app.kubernetes.io/name` is used to display the application’s pods in the SoFy Solution Console. Label `app.kubernetes.io/instance` is used to select all Kubernetes resources from the SoFy solution install. You are responsible to make sure Kubernetes resources from any third-party dependency charts also contain and use these labels properly.


```
app.kubernetes.io/name: {{ include "my-helm-chart.name" . }}
app.kubernetes.io/instance: {{ .Release.Name }}
```

**Note:** The following labels have been superseded by `app.kubernetes.io/name` and `app.kubernetes.io/instance`. Do not use them.

```yaml
name:  {{ include "my-helm-chart.name" . }}
release: {{ .Release.Name }}
```

If you used `helm create my-chart-name` as the template or started your chart, you should be all set.

#### 6. Getting Sofy Domain in a Container

There are scenarios where a container (inside a pod) needs to know the name of the Sofy sub-domain. For example, when constructing the host name for a particular Kubernetes service. This can be achieved in either of the following ways:

-	Using the `global.domain` variable (present in `values.yaml`) that’s set for each solution deployed in a SoFy sandbox, or a custom domain set at solution install time, outside of the sandbox.

It can be used as follows:

 ```yaml
      containers:
        - name: example
          image: "busybox:latest"
          env:
            - name: MY_SUB_DOMAIN
              value: "{{ .Values.global.domain }}"
  ```

- Using ConfigMap `{{ .Release.Name}}-domain` to get the sofy-domain.

  Every deployed solution comes with a ConfigMap `{{ .Release.Name }}-domain` that has variables `HOST` and `HOST_PROTOCOL`, which are configured dynamically when a solution is installed.
  
  The `HOST` variable will have a Sofy sub-domain and `HOST_PROTOCOL` will be `https` (since all Sofy solution have TLS enabled.)

  This ConfigMap can be used in a container as follows:

  ```yaml
      containers:
        - name: example
          image: "busybox:latest"
          env:
            - name: MY_SUB_DOMAIN
              valueFrom:
                configMapKeyRef:
                  name: {{ .Release.Name }}-domain
                  key: HOST
  ```

  This will return the "sub domain". For example the SoFy KeyCloak discovery endpoint could then be retrieved with the below by inserting the `env.SOLUTION_DOMAIN` into the rest of the known (or constructed) URL.
  
  ```yaml
  discoveryEndpointUrl="https://sofy-kc.${env.MY_SUB_DOMAIN}/auth/realms/sofySolution/.well-known/openid-configuration"
  ```

  This is very helpful in scenarios when a solution is deployed in a private cluster (other than Sofy sandbox), where the Sofy's Ambassador (load balancer) will not have a sub-domain set and instead use dynamic IP (if available) to create a subdomain of type `**external-ip**.nip.io`. Thus, the container can get the dynamic domain, and not depend on a solution admin to supply it.

  > **Note:** Due to the dynamic nature of assigning an external IP to a service in a Kubernetes cluster, which varies by cloud provider, the ConfigMap may not immediately have the environment variable available. In that case, a pod which has a container trying to retrieve the environment variable will not start up until the ConfigMap is properly configured.

#### 7. Kubernetes Resource Naming Best Practice

It is recommended that Kubernetes resources like Pods, PVC's, and Services use the product name and release name as part of their naming scheme. Example: `{{ .Release.Name }}-onedb` 

This is done so that these resources don't conflict with other products/services inside the same SoFy solution. This also improves user experience and debugging because of the improved visibility on which resource belongs to which services. 

## Windows Images

SoFy sandboxes run on GKE clusters.  GKE windows server nodes support running **Windows Server version 2019 LTSC (Long Term Support Channel) images**. SoFy sandboxes do not support running any Windows images from SAC (Semi-Annual Channel) because [GKE's frequent and arbitrary stop of cluster nodes creation when a SAC version ends its support lifecycle.](https://cloud.google.com/kubernetes-engine/docs/how-to/creating-a-cluster-windows)

Both Windows Server Core and Nano Server from LTSC can be used as a base image for your containers. See [Windows Image version compatibility](https://cloud.google.com/kubernetes-engine/docs/how-to/creating-a-cluster-windows#version_compatibility) and [Version mapping by GKE](https://cloud.google.com/kubernetes-engine/docs/how-to/creating-a-cluster-windows#version_mapping).

The following windows base images have been tested on SoFy sandbox cluster:

- `FROM mcr.microsoft.com/windows/servercore:ltsc2019`
- `FROM mcr.microsoft.com/windows:10.0.17763.1817`

SoFy's production sandbox will start to support LTSC Windows Server version 2019 before the end of May 2021. Please let SoFy team know if your product have different needs.

### Use Node Selectors and Tolerations

While using Windows Image, it is important that the pod gets sheduled in appropriate Windows node pool. To ensure that the Windows pods get sheduled in the correct node pool you need to add following node selectors and node tolerations.

In **values.yaml**:

```yaml
nodeSelector:
  beta.kubernetes.io/os: windows
  node.kubernetes.io/windows-build: "10.0.17763"
tolerations: 
- key: "node.kubernetes.io/os"
  operator: "Equal"
  value: "windows"
  effect: "NoSchedule"
```

In **your pod specification** of your deployment.yaml or statefulset.yaml:
```yaml
{{- with .Values.nodeSelector }}
      nodeSelector:
{{ toYaml . | indent 8 }}
{{- end }}
{{- with .Values.affinity }}
{{- with .Values.tolerations }}
      tolerations:
{{ toYaml . | indent 8 }}
    {{- end }}
```

In the above node selectors, the first selector is the label for all the nodes with Windows OS and the second selector is the label for all the nodes with Windows LTSC server. The `tolerations` allows your windows pods to be schedulable on windows nodes with the "windows" taint, but other pods w/o this toleration will not be schedulable on such nodes.

### Windows Pods Examples

1. [snoop windows](https://github01.hclpnp.com/kubernetes/sofy-catalog-content/tree/master/GA/snoop-windows-ltsc2019)

2. [Unica Discover](https://github01.hclpnp.com/kubernetes/sofy-catalog-content/tree/master/GA/unica-discover) uses windows image. 

## Tips and Tricks

- If you are starting from scratch, use `helm create my-chart-name` to get a working sample of a helm chart with best practices. This is a great starting point and satisfies all common K8s and helm charts requirements above.

## Also reference

- [Products Checklist for SoFy](https://hclo365.sharepoint.com/:p:/r/sites/SoFyProductContentCollaboration/Shared%20Documents/General/Darwin%20SoFy%20Onboarding/SoFy%20Onboarding%20Checklists/HCL-Products-SoFy-readiness-checklist.pptx?d=wa8a4906719354246bd335be2d217ad1d&csf=1&web=1&e=pdL0g8)
- [Demo Packs Checklist for SoFy](https://hclo365.sharepoint.com/:p:/r/sites/SoFyProductContentCollaboration/Shared%20Documents/General/Darwin%20SoFy%20Onboarding/SoFy%20Onboarding%20Checklists/HCL-Demos-SoFy-readiness-checklist.pptx?d=w5610fe7e837b433ebadc905daeba6719&csf=1&web=1&e=sUxapu)
- Live [Cloud Native Knowledge Base](https://github01.hclpnp.com/pages/kubernetes/knowledge-base/)
