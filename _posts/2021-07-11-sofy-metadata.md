---
title: SoFy Metadata
description: Key to getting SoFy working properly is making the necessary adjustments to your Helm chart's metadata.
layout: home
---
- [Generic Metadata](#generic-metadata)
- [Demo Packs](#demo-packs-metadata)
- [Demo Docs & Proxies](#demo-docs-and-proxy-metadata)
- [Hybrid Mode](#hybrid-mode-metadata)
- [Incompatibilities](#incompatibilities-metadata)

**Note**: This document will continue evolving with the HCL software cloud native journey.

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
