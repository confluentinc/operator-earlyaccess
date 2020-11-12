Confluent Operator Early Access
===============================

The next version of Confluent Operator introduces a new Kubernetes-native
interface, including:

* First class Kubernetes custom resource definitions (CRDs)
  
  You do not need Helm to install Confluent Platform components.
  
* Config overrides for all Confluent components
* Auto generated certificates
* Kubernetes tolerations, pod/node affinity support

Get early access by registering interest here: <link to form>

The assumption is that you understand Kubernetes and Confluent Platform concepts:

* Learn about `Kubernetes <https://kubernetes.io/docs/home>`__`.
* Lean about `Confluent Platform <https://docs.confluent.io>`__.

To use this Confluent Operator Early Access, you’ll need:

* A Kubernetes cluster - any CNCF conformant version
* Helm 3 installed on your local machine
* Kubectl installed on your local machine

This Confluent Operator Early Access is compatible with:

* Confluent Platform 1.6.0

==================
Scenario workflows
==================

This documentation is organized by scenario workflows. Clone this repo to get the files needed for each workflow:

::

  git clone git@github.com:confluentinc/operator-earlyaccess.git

* Deploy simple non-secure Confluent Platform
* Configure external access through Load Balancer
* Configure external access through NodePorts
* Deploy encryption and authentication for Confluent Platform
* Deploy RBAC for Confluent Platform

  * Configure RBAC for Confluent Platform
  * Configure and deploy OpenLDAP
  * Bootstrap role bindings
  * Validate with sample producer and required rolebindings

* Declaratively manage topics
  
  * Configure Topic Operator
  * Create a topic
  * Edit a topic
  * Delete a topic

==================================
Configure Early Access credentials
==================================

The Confluent Operator itself is packaged as a Helm Chart. 

#. For this Early Access program, you will have received an API key (associated with your email address) to the Confluent JFrog Artifactory.

#. Save the API key and your email for easy access while running this tutorial:

   ::

     export USER=<your email address>
     export EMAIL=<your email address>
     export APIKEY=<your API key>

#. Create a Kubernetes secret with the registry credentials:

   ::
   
     kubectl create secret docker-registry confluent-registry \
       --docker-server=confluent-docker.jfrog.io \   
       --docker-username=$USER \
             --docker-password=$APIKEY \
             --docker-email=$EMAIL

#. Add a Helm repo:

   ::

     helm repo add confluentinc \   
       https://confluent.jfrog.io/confluent/helm-cloud \
       --username $USER \
       --password $APIKEY

============================================
Download Confluent Operator tutorial package
============================================

Download the tutorial package:

::

  git clone git@github.com:confluentinc/operator-earlyaccess.git


