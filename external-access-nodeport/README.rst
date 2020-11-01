Deploy Confluent Platform with NodePort
=======================================

In this scenario workflow, you'll set up a simple non-secure (no authentication, authorization, or encryption) Confluent Platform component clusters with the Kubernetes NodePort service for external access.

With the NodePort services, Kubernetes will allocate a port on all nodes of the Kubernetes cluster and will make sure that all traffic to this port is routed to the pods.

The goal for this scenario is for you to:
- Set up a complete Confluent Platform on Google Kubernetes Engine for external access using Kubernetes NodePort.
- Configure a producer to generate sample data.

