Prepare for Confluent on Kubernetes Deployment
==============================================

To prepare for a deployment, it's useful to have a plan along the following dimensions.

- Kubernetes Infrastructure
- Cluster Sizing
- Docker Registry
- Storage
- Kubernetes Security
- Confluent Platform Security
- Networking

=========================
Kubernetes Infrastructure
=========================

Confluent Operator supports Kubernetes versions 1.16 to 1.19 with any Cloud Native Computing Foundation (CNCF) conformant offering. 
A full list of CNCF conformant Kubernetes offerings can be found `here <https://docs.google.com/spreadsheets/d/1LxSqBzjOxfGx3cmtZ4EbB_BGCxT_wlxW_xgHVVa23es/edit#gid=0>`__.

==============
Cluster Sizing
==============

Decide the Confluent components needed and the sizing (RAM, CPU and storage) for each component.

For sizing recommendations, use the following:

- https://docs.confluent.io/operator/current/co-plan.html#sizing-recommendations
- https://eventsizer.io/

===============
Docker Registry
===============

Confluent Operator pulls Confluent Docker images from a Docker registry and deploys those on to your Kubernetes cluster.

By default, Confluent Operator deploys publicly-available Docker images hosted on Docker Hub from the ``confluentinc`` repositories.

You can choose to use your own Docker registry and repositories. In that case, you'll need to pull the images from the Confluent repositories 
and upload to your Docker registry repositories.

=======
Storage
=======

You'll need to provide persistent storage for the following:

- Kafka
- Zookeeper

Kafka can be provisioned with (1) persistent storage volumes and (2) tiered storage. 

For (1) - Confluent supports the use of `Storage Classes <https://kubernetes.io/docs/concepts/storage/dynamic-provisioning/>`__ to provision 
persistent storage volumes. 

Kafka requires Block level storage solutions - AWS EBS, Azure Disk, GCE Disk, Ceph RBD, Portworx are examples.

For (2) - Confluent supports various Object Storage solutions:

- AWS S3
- GCP GCS
- Pure Storage FlashBlade

For Zookeeper, use the same storage solution as for Kafka in (1).

===================
Kubernetes Security
===================

With Kubernetes RBAC and namespaces, you can deploy in either of two ways:

- Provide Confluent Operator with access to provision and manage Confluent resources across all namespaces in the Kubernetes cluster
- Provide Confluent Operator with access to provision and manage Confluent resources in one specific namespace

==================
Confluent Security
==================

Security for a Confluent deployment covers the following dimensions:

- Authentication
- Authorization
- Network Encryption
- Configuration Secrets

For production deployments, Confluent recommends: `secure configuration for production: <../production-secure-deploy>`_

You can choose other options for each dimension.

==========
Networking
==========

The Confluent services can be exposed for access to users and client applications that can either be:
1. In the internal Kubernetes network
2. External to the Kubernetes network

To expose externally, you have a few options:

- Load Balancers

  - For Kafka: A Layer 4 load balancer that supports TLS passthrough
  - For other Confluent services with HTTP endpoints: A Layer 4/7 load balancer

- Ingress Controller

  - For Kafka: An Ingress controller that supports TLS passthrough and routes TCP traffic

When exposing services externally, Confluent services will be made available at externally resolvable domain names. You'll need
to manage DNS entries to facilitate these domains.
