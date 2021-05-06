Use Helm to deploy the Confluent Operator
==========================================

`Helm <https://helm.sh/>`_ is a package manager for Kubernetes. Helm packages are `charts <https://helm.sh/docs/topics/charts/>`_ 
- a collection of files that describe a related set of Kubernetes resources.
The Confluent Operator is packaged as a Helm chart.

The current stable version of Helm is v3. Confluent Operator 2.0 supports Helm 3. `Helm 2 has been deprecated <https://helm.sh/blog/helm-v2-deprecation-timeline/>`_.

In this section, we'll cover common Helm usage commands for the Confluent Operator.

==================================
Manage Helm packages in repository
==================================

A Helm chart repository is a location where packaged charts are stored and shared. The Confluent Operator Helm chart is 
stored at https://confluent.jfrog.io/confluent/helm-early-access-operator-2

Add the Confluent chart repository to your Helm repository:

::

   helm repo add confluentinc_earlyaccess \   
       https://confluent.jfrog.io/confluent/helm-early-access-operator-2 \
       --username $USER \
       --password $APIKEY
  
View all Helm charts in your Helm repository:

::

   helm repo list

Update all Helm charts in repository to get latest versions:

::
   helm repo update

Remove a specific Helm chart from your Helm repository

::

   helm repo remove confluentinc_earlyaccess

=============================
Deploy and Manage Helm charts
=============================

You deploy Helm charts to a release on your Kubernetes.

::

   helm install <release> <chart>
   

There are two ways to deploy Confluent Operator chart to your Kubernetes:

::

   # Install for the first time to release name "operator"
   helm install operator confluentinc_earlyaccess/confluent-for-kubernetes

   # Upgrade to latest version, or install if the release does not exist
   helm upgrade --install operator confluentinc_earlyaccess/confluent-for-kubernetes

Install a specific version of the Helm chart:

::

   helm install operator confluentinc_earlyaccess/confluent-for-kubernetes --version 0.72.0

View all installed Helm releases on your Kubernetes:

::

   helm list

Delete the Helm release from your Kubernetes:

::

   helm delete operator

View the Helm docs for all available Helm commands: https://helm.sh/docs/helm/helm/
