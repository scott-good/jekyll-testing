---
layout: home
title: Helm Chart Requirements
description: Before uploading to SoFy, your Helm chart needs to pass seven important tests
---

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