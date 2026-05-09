---
title: "When ConfigMaps Hit Limits: Migrating to CRDs"
url: "https://kubex.ai/blog/when-configmaps-hit-limits-migrating-to-crds/"
date: "2026-02-23"
author: "Julien Semaan"
feed_url: "https://www.densify.com/blog/feed/"
---
Over the past few years, Kubex has evolved from a cloud optimization product into a Kubernetes-centric solution, shifting its focus from cost and waste visibility to fully automated resource optimization. As that evolution happened, one of the earliest design decisions we had made began to show its limits: how the product was configured. Our original automation component, responsible for making optimization decisions across customer clusters, was configured either through Helm chart values or by manually editing a ConfigMap.
