---
title: "Kubernetes GPU Scheduling for MLOps and GPU Sharing"
url: "https://kubex.ai/blog/kubernetes-gpu-scheduling/"
date: "2026-08-06"
author: "David Chase"
feed_url: "https://kubex.ai/blog/feed/"
---
Introduction The default Kubernetes scheduler was built for stateless services: web servers, APIs, databases. It schedules a pod, checks that a node has enough of whatever resources were requested, and binds it. For CPU and memory, that model works fine.
