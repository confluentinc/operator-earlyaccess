Confluent Operator Early Access
===============================

The next version of Confluent Operator offers a Kubernetes-native experience, including:

* First class Kubernetes custom resource definitions (CRDs)
* Config overrides for all Confluent components
* Auto generated certificates
* Kubernetes tolerations, pod/node affinity support

Check out the product in action: `Kubernetes-Native DevOps with Confluent Operator <https://www.youtube.com/watch?v=lqoZSs_swVI&feature=youtu.be>`_

Get early access by registering interest here: `Confluent Operator Early Access Registration <https://events.confluent.io/confluentoperatorearlyaccess>`_

==================
Scenario workflows
==================

The documentation is organized as scenario workflows. Clone this repo to get the files needed for each workflow:

::

  git clone git@github.com:confluentinc/operator-earlyaccess.git

* `Deploy simple non-secure Confluent Platform <./quickstart-deploy>`_
* `Deploy Confluent Platform with encryption and authentication <./secure-authn-encrypt-deploy>`_
* `Deploy Confluent Platform with RBAC authorization <./cp-rbac-deploy>`_
* `Deploy Confluent Platform with external access through Load Balancer <external-access-load-balancer-deploy>`_
* `Deploy Confluent Platform with external access through NodePorts <external-access-nodeport-deploy>`_

.. _ea-credentials:

==============
Pre-requisites
==============

This Confluent Operator Early Access is compatible with Confluent Platform 1.6.0.

To use this Confluent Operator Early Access, you’ll need:

* A Kubernetes cluster - any CNCF conformant version
* Helm 3 installed on your local machine
* Kubectl installed on your local machine

=============================
Set up the Kubernetes cluster
=============================

Set up the Kubernetes cluster for this tutorial.

#. Add or get access to a Kubernetes cluster.

#. Create the namespace and set it to the current namespace. In this tutorial, we will deploy Confluent Platform in the ``confluent`` namespace.

   ::
   
     kubectl create namespace confluent
   
   ::

     kubectl config set-context --current --namespace=confluent

==================================
Configure Early Access credentials
==================================

#. For this Early Access program, you will have received an API key (associated with your email address) to the Confluent JFrog Artifactory.

#. Save the API key and your email for easy access while running this tutorial:

   ::

     export USER=<your email address>
     export EMAIL=<your email address>
     export APIKEY=<your API key>

#. Create a Kubernetes secret with the registry credentials:

   ::
   
     kubectl create secret docker-registry confluent-registry \
       --docker-server=confluent-docker-internal-early-access-operator-2.jfrog.io \   
       --docker-username=$USER \
             --docker-password=$APIKEY \
             --docker-email=$EMAIL

#. The Confluent Operator itself is packaged as a Helm Chart. Add a Helm repo:

   ::

     helm repo add confluentinc \   
       https://confluent.jfrog.io/confluent/helm-cloud \
       --username $USER \
       --password $APIKEY

   :: 
   
     helm repo update
     
.. _download_tutorials:

============================================
Download Confluent Operator tutorial package
============================================

Download the tutorial package from the Git Hub repo:

::

  git clone git@github.com:confluentinc/operator-earlyaccess.git
