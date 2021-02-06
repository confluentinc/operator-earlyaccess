Deploy Connectors and KsqlDB using managed Confluent Cloud Kafka
================================================================

You can use Confluent Operator to deploy and manage Connectors and KsqlDB against a Confluent Cloud Kafka and Schema Registry.

==================================
Set the current tutorial directory
==================================

Set the tutorial directory for this tutorial under the directory you downloaded
the tutorial files:

::
   
  export TUTORIAL_HOME=<Tutorial directory>/ccloud-integration
  
=========================
Deploy Confluent Operator
=========================

The assumption is that you’ve set up Early Access credentials following `the
instruction
<https://github.com/confluentinc/operator-earlyaccess/blob/master/README.rst>`__.

#. Install Confluent Operator using Helm:

   ::

     helm upgrade --install operator confluentinc_earlyaccess/confluent-operator --set image.registry=confluent-docker-internal-early-access-operator-2.jfrog.io
  
#. Check that the Confluent Operator pod comes up and is running:

   ::
     
     kubectl get pods


============================
Deploy configuration secrets
============================

Provide a Root Certificate Authority
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Confluent Operator provides auto-generated certificates for Confluent Platform
components to use for inter-component TLS. You'll need to generate and provide a
Root Certificate Authority (CA).

#. Generate a CA pair to use in this tutorial:

   ::

     openssl genrsa -out $TUTORIAL_HOME/ca-key.pem 2048
    
   ::

     openssl req -new -key $TUTORIAL_HOME/ca-key.pem -x509 \
       -days 1000 \
       -out $TUTORIAL_HOME/ca.pem \
       -subj "/C=US/ST=CA/L=MountainView/O=Confluent/OU=Operator/CN=TestCA"

#. Create a Kuebernetes secret for inter-component TLS:

   ::

     kubectl create secret tls ca-pair-sslcerts \
       --cert=$TUTORIAL_HOME/ca.pem \
       --key=$TUTORIAL_HOME/ca-key.pem

Provide authentication credentials
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Confluent Cloud provides you an API key for both Kafka and Schema Registry.
Configure Confluent Operator to use the API key when setting up Connect and KsqlDB to connect.

Create a Kubernetes secret object for Confluent Cloud Kafka access.
This secret object contains file based properties. These files are in the
format that each respective Confluent component requires for authentication
credentials:

   ::
   
     kubectl create secret generic cloud-plain \
      --from-file=plain.txt=$TUTORIAL_HOME/creds-client-kafka-sasl-user.txt

   ::
   
     kubectl create secret generic cloud-sr-access \
      --from-file=basic.txt=$TUTORIAL_HOME/creds-schemaRegistry-user-mine.txt
   
   ::
   
     kubectl create secret generic control-center-user \
      --from-file=basic.txt=$TUTORIAL_HOME/creds-control-center-users.txt

=========================
Deploy Confluent Platform
=========================

Edit the ``confluent-platform.yaml`` deployment YAML, and add your respective Confluent Cloud URLs in the following places:

- ``<cloudSR_url>``
- ``<cloudKafka_url>``



#. Deploy Confluent Platform with the above configuration:

   ::

     kubectl apply -f $TUTORIAL_HOME/confluent-platform.yaml

#. Check that all Confluent Platform resources are deployed:

   ::
   
     kubectl get pods

========
Validate
========

Validate in Control Center
^^^^^^^^^^^^^^^^^^^^^^^^^^

Use Control Center to monitor the Confluent Platform, and see the created topic
and data.

#. Set up port forwarding to Control Center web UI from local machine:

   ::

     kubectl port-forward controlcenter-0 9021:9021

#. Browse to Control Center and log in as the ``admin`` user with the ``Developer1`` password:

   ::
   
     https://localhost:9021

=========
Tear down
=========

::

  kubectl delete -f $TUTORIAL_HOME/confluent-platform.yaml

::

  kubectl delete secrets cloud-plain cloud-sr-access control-center-user

::

  kubectl delete secret ca-pair-sslcerts

::

  helm delete operator
