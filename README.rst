Confluent Operator Early Access
===============================

The next version of Confluent Operator offers a Kubernetes-native experience, including:

* First class Kubernetes custom resource definitions (CRDs)
* Config overrides for all Confluent components
* Auto generated certificates
* Kubernetes tolerations, pod/node affinity support

Check out the product in action: https://www.youtube.com/watch?v=lqoZSs_swVI&feature=youtu.be

Get early access by registering interest here: https://events.confluent.io/confluentoperatorearlyaccess

==================
Scenario workflows
==================

The documentation is organized as scenario workflows. Clone this repo to get the files needed for each workflow:

::

  git clone git@github.com:confluentinc/operator-earlyaccess.git

* Deploy simple non-secure Confluent Platform
* Deploy encryption and authentication for Confluent Platform
* Deploy rbac authorization for Confluent Platform
* Configure external access through Load Balancer
* Configure external access through NodePorts

.. _ea-credentials:

==============
Pre-requisites
==============

This Confluent Operator Early Access is compatible with Confluent Platform 1.6.0.

To use this Confluent Operator Early Access, you’ll need:

* A Kubernetes cluster - any CNCF conformant version
* Helm 3 installed on your local machine
* Kubectl installed on your local machine

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
       --docker-server=confluent-docker.jfrog.io \   
       --docker-username=$USER \
             --docker-password=$APIKEY \
             --docker-email=$EMAIL

#. The Confluent Operator itself is packaged as a Helm Chart. Add a Helm repo :

   ::

     helm repo add confluentinc \   
       https://confluent.jfrog.io/confluent/helm-cloud \
       --username $USER \
       --password $APIKEY

.. _download_tutorials:

============================================
Download Confluent Operator tutorial package
============================================

Download the tutorial package from the Git Hub repo:

::

  git clone git@github.com:confluentinc/operator-earlyaccess.git


