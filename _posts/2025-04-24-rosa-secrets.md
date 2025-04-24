---
layout: post
title:  "On ROSA: ESO, Reloader, and Secrets Manager"
date:   2025-04-24 08:14:04 -0700
categories: rosa
---

## Context

My students at a leading U.S.-based insurance company are mandated to migrate their applications from PCF and TP2 to AWS. To accelerate this migration, the company selected Red Hat OpenShift on AWS (ROSA) as their **lift** target. From there, teams can **shift**, rearchitecting for cloud-native by leveraging AWS services.

To meet tight deadlines, my team was contracted to develop and deliver training that would equip their developers with the skills needed to successfully adopt the technologies abstracted by a GitOps-based migration to ROSA. These include GitLab CI/CD, Docker, Kubernetes, AWS IAM and Secrets Manager, ROSA, External Secrets Operator (ESO), Reloader, and ArgoCD. While the cloud enablement team provided thorough, cookbook-style documentation, they were overwhelmed with support requests from developers who lacked the foundational knowledge to resolve the inevitable issues that arise when deploying bespoke applications to ROSA via CI/CD.

That’s where our team came in. We specialize in developing and delivering immersive, hands-on training tailored for organizations undergoing cloud transformations.

Specifically, I worked alongside another developer and an engagement manager for three months to learn the client’s stack and replicate it in our own environment. We were not granted access to their infrastructure or network, so we had to recreate everything—or find suitable alternatives. Our team consisted of myself, a full-stack JavaScript developer with experience in Docker, Kubernetes, and AWS, and my colleague, a former DBA and Java developer, also familiar with Kubernetes and AWS. Together, we deployed a ROSA cluster for the first time, learned ESO from scratch, installed and configured ArgoCD, wrote custom CI/CD components to sync applications, and authored all example apps, labs, and technical documentation.

Throughout the development phase, we collaborated closely with stakeholders from the client’s cloud enablement team to ensure the course would meet their goals. But it was only after we began teaching and received direct feedback from the developers that we were able to adapt the course to better suit their needs. Within two months, our course achieved an NPS score of 100 after training nearly 150 developers—representing about 60% of the teams involved in ROSA migrations.

While I was already familiar with Docker and Kubernetes, the most valuable part of this project for me was deepening my understanding of **External Secrets Operator** and how it integrates with AWS IAM.

## A need for Secrets Management

Enterprise-grade Kubernetes clusters must follow [zero-trust principles](https://aws.amazon.com/security/zero-trust/) when accessing external services. Native Kubernetes Secrets aren’t enough—they must be rotated, and access must be tightly controlled. This is where [IAM Roles for Service Accounts (IRSA)](https://docs.aws.amazon.com/eks/latest/userguide/iam-roles-for-service-accounts.html) come into play. IRSA allows a Kubernetes pod to assume an AWS IAM role via federated access through an OIDC provider. Once assumed, the role can perform actions as defined by its attached IAM policy.

To ease students into this complex setup, we started with a simplified scenario. My colleague deployed a PostgreSQL database on EC2 and manually stored credentials in AWS Secrets Manager. Students received IAM user credentials with minimal permissions to fetch the secret, which they then used in minikube clusters with ESO. No OIDC, no custom service accounts—just enough to understand the core concept. However, after one delivery, we scrapped this approach. Teaching a bad practice, even with a disclaimer, felt counterproductive—especially since our next modules demonstrated ESO with IRSA on the ROSA cluster. That felt like enough.

We reminded students they only needed to deeply understand these moving parts if they were manually onboarding secrets or integrating with other AWS services. In reality, the client had GUI tools that automated the onboarding process—creating secrets, IAM roles, trust policies, and permissions behind the scenes. These abstractions were designed to accelerate adoption, but migration bottlenecks still existed, particularly with database migrations involving snapshotting, user management, and secrets rotation.

In every Kubernetes class I’ve taught, at least half the room goes silent when the jargon hits. Add AWS IAM to the mix, and the confusion multiplies. To combat this, I rely on live diagramming to visually explain trust relationships and data flows. Once students see how these components connect, the questions start rolling in—especially around how networking and secrets rotation works.

Currently, our course demonstrates ESO with IRSA and static secrets in Secrets Manager, but I’m excited to be getting more requests to go deeper and show **automatic secrets rotation** in action. Developers are also asking how updates to ConfigMaps or Secrets trigger application rollouts—which is where [Reloader](https://github.com/stakater/Reloader) comes in.

So tomorrow, I’ll be implementing this flow on our ROSA cluster. Reloader is installed, an RDS instance is provisioned, and I’ll be authoring the Lambda function and IAM resources to demonstrate rotating secrets. Since we may need to tear down the cluster between cohorts, I’ll also be codifying everything in Terraform and storing it in GitLab with its Terraform integration for repeatable setups.
