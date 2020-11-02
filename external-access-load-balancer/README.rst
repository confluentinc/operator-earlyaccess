Deploy Confluent Platform with Load Balancer
============================================

In this scenario workflow, you'll set up a simple non-secure (no authentication, authorization, or encryption) Confluent Platform component clusters with the Kubernetes service type LoadBalancer for external access.

The goal of this scenario is to learn how to:

- Set up a complete Confluent Platform on Google Kubernetes Engine for external access using Kubernetes LoadBalancer.
- Configure a producer topic to generate sample data.

To complete this scenario, you'll follow these steps:

#. Deploy Confluent Operator
#. Deploy Confluent Platform
#. Deploy a Producer application
#. Tear down Confluent Platform

