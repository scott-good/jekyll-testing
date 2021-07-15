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

## Recent Blog Posts

Lorem ipsum dolor sit amet, consectetuer adipiscing elit, sed diam nonummy nibh euismod tincidunt ut laoreet dolore magna aliquam erat volutpat. Ut wisi enim ad minim veniam, quis nostrud exerci tation [ullamcorper suscipit lobortis](https://google.com) nisl ut aliquip ex ea commodo consequat. Duis autem vel eum iriure dolor in hendrerit in vulputate velit esse molestie consequat, vel illum dolore eu feugiat nulla facilisis at vero eros et accumsan et iusto odio dignissim qui blandit praesent luptatum zzril delenit augue duis dolore te feugait nulla facilisi.

<img src="{{ "assets/images/landscape-architecture-fun-facts-outside-productions-blog-980x551.jpg" | relative_url }}" class='inline-image' />

Li Europan lingues es membres del sam familie. Lor separat existentie es un myth. Por scientie, musica, sport etc, li tot Europa usa li sam vocabularium. Li lingues differe solmen in li grammatica, li pronunciation e li plu commun vocabules. Omnicos directe al desirabilita; de un nov lingua franca: on refusa continuar payar custosi traductores. It solmen va esser necessi far uniform grammatica, pronunciation e plu sommun paroles.