Configure Rack Awareness
=========================

In this workflow scenario, you'll deploy Confluent Platform to a multi-zone Kubernetes cluster, with Kafka rack awareness configured.

Each zone will be treated as a rack. Kafka brokers will be round-robin scheduled across the zones. 
Each Kafka broker will be configured with it's broker.rack setting to the corresponding zone it's scheduled on.

Read more about Kafka rack awareness: `Confluent Docs <https://docs.confluent.io/platform/current/kafka/post-deployment.html#balancing-replicas-across-racks>`__.

Before you begin this tutorial:

* `Set up the prerequisites <https://github.com/confluentinc/operator-earlyaccess#pre-requisites>`__.

* `Create the namespace for the tutorials <https://github.com/confluentinc/operator-earlyaccess#set-up-the-kubernetes-cluster>`__.

* `Configure the Early Access credentials <https://github.com/confluentinc/operator-earlyaccess#configure-early-access-credentials>`__.

* `Clone the tutorial repo <https://github.com/confluentinc/operator-earlyaccess#download-confluent-operator-tutorial-package>`__.

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

The Kubernetes cluster must have node labels set for the fault domain. The nodes in each zone should have the same node label.

Check the Node Labels for ``topology.kubernetes.io/zone``:

::

  kubectl get nodes --show-labels | grep 'zone'

Configure your Kubernetes cluster with a service account that is configured with a clusterrole/role that provides 
get/list access to both the pods and nodes resources.
This is required as Kafka pods will curl kubernetes api for the node it is scheduled on using the mounted serviceAccountToken.

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

     helm upgrade --install operator confluentinc_earlyaccess/confluent-operator \
       --set image.registry=confluent-docker-internal-early-access-operator-2.jfrog.io
  
#. Check that the Confluent Operator pod comes up and is running:

   ::
     
     kubectl get pods

=======================================
Configure and Deploy Confluent Platform
=======================================

#. Configure rack awareness - see the example file `rackawareness/confluent-platform.yaml`

   ::

     ...
     kind: Kafka
     ...
     spec:
       replicas: 6
       rackAssignment:
         nodeLabels:
         - topology.kubernetes.io/zone # <-- The node label that your Kubernetes uses for the rack fault domain
       podTemplate:
        serviceAccountName: kafka # <-- The service account with the needed clusterrole/role
       oneReplicaPerNode: true # <-- Ensures that only one Kafka broker per node is scheduled
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

     