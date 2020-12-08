Configure Rack Awareness
=========================

In this workflow scenario, you'll deploy Confluent Platform to a multi-zone Kubernetes cluster,
with Kafka rack awareness configured.

==================================
Set the current tutorial directory
==================================

Set the tutorial directory for this tutorial under the directory you downloaded
the tutorial files:

::
   
  export TUTORIAL_HOME=<Tutorial directory>/rackawareness

===========================================================
Pre-requisites: Node Labels and Service Account Rolebinding
===========================================================

The Kubernetes cluster must have Node Labels set. Each zone should have a node label.

Check the Node Labels:

   ::

     kubectl get nodes --show-labels

The Kubernetes cluster must have a service account that is configured with a clusterrole/role that provides 
get/list access to both the pods and nodes resources.

Kafka pods will curl kubernetes api for the node it is scheduled using the mounted serviceAccountToken.

   ::

     kubectl apply -f rackawareness/service-account-rolebinding.yaml

=========================
Deploy Confluent Operator
=========================

The assumption is that you’ve set up Early Access credentials following `the
instruction
<https://github.com/confluentinc/operator-earlyaccess/blob/master/README.rst>`__.

#. Install Confluent Operator using Helm:

   ::

     helm upgrade --install operator confluentinc/confluent-operator --set image.registry=confluent-docker-internal-early-access-operator-2.jfrog.io
  
#. Check that the Confluent Operator pod comes up and is running:

   ::
     
     kubectl get pods

=========================
Configure and Deploy Confluent Platform
=========================

#. Configure rack awareness - see the example file `rackawareness/confluent-platform.yaml`

   ::

     ...
     kind: Kafka
     ...
     spec:
       replicas: 6
       rackAssignment:
         nodeLabels: failure-domain.beta.kubernetes.io/zone
       ...

#. Deploy the Confluent Platform

   ::

  kubectl apply -f $TUTORIAL_HOME/confluent-platform.yaml

#. Check that all Confluent Platform resources are deployed:

   ::
   
     kubectl get confluent

#. Get the status of any component. For example, to check Kafka:

   ::
   
     kubectl describe kafka

     