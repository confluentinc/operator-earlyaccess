Deploy Confluent Platform with Static Host-based Routing
========================================================

In this scenario workflow, you'll set up Confluent Platform component clusters
with the static host-based routing to enable external clients to access
Kafka.

Before you begin this tutorial:

* `Set up the prerequisites <https://github.com/confluentinc/operator-earlyaccess#pre-requisites>`__.

* `Create the namespace for the tutorials <https://github.com/confluentinc/operator-earlyaccess#set-up-the-kubernetes-cluster>`__.

* `Configure the Early Access credentials <https://github.com/confluentinc/operator-earlyaccess#configure-early-access-credentials>`__.

* `Clone the tutorial repo <https://github.com/confluentinc/operator-earlyaccess#download-confluent-operator-tutorial-package>`__.
 
To complete this tutorial, you'll follow these steps:

#. Set up the Kubernetes cluster for this tutorial.

#. Set the current tutorial directory.

#. Deploy Confluent Operator.

#. Deploy Confluent Platform.

#. Deploy the producer application.

#. Tear down Confluent Platform.

===========================
Set up a Kubernetes cluster
===========================

Set up a Kubernetes cluster for this tutorial, and save the Kubernetes cluster
domain name. 
 
In this document, ``$DOMAIN`` will be used to denote your Kubernetes cluster
domain name.
  
::

  export DOMAIN=<Your Kubernetes cluster domain name>

==================================
Set the current tutorial directory
==================================

Set the tutorial directory for this tutorial under the directory you downloaded
the tutorial files:

::
   
  export TUTORIAL_HOME=<Tutorial directory>/external-access-static-host-based

=========================
Deploy Confluent Operator
=========================

#. Install Confluent Operator using Helm:

   ::
   
     helm upgrade --install operator confluentinc_earlyaccess/confluent-operator \
       --set image.registry=confluent-docker-internal-early-access-operator-2.jfrog.io

#. Check that the Confluent Operator pod comes up and is running:

   ::
   
     kubectl get pods
      
===============================
Generate and Deploy TLS Secrets
===============================

This tutorial uses mutual TLS (mTLS) for encryption and authentication between
Confluent Server (inside the Kubernetes cluster) and Kafka clients (outside the Kubernetes cluster).
In production, you would need to provide your own valid certificates,
but for this tutorial, you will generate your own. You will create:

* ``cacerts.pem`` -- Certificate authority (CA) certificate
* ``privkey.pem`` -- Private key for Confluent Server
* ``fullchain.pem`` -- Certificate for Confluent Server
* ``privkey-client.pem`` -- Private key for Kafka client
* ``fullchain-client.pem`` -- Certificate for Kafka client
* ``client.truststore.p12`` -- Truststore for Kafka client
* ``client.keystore.p12`` -- Keystore for Kafka client

Notice that you will not generate the keystore or truststore for Confluent Server.
Confluent Operator will generate those for Confluent Server from
``cacerts.pem``, ``privkey.pem``, and ``fullchain.pem``.

#. 

#. Create a Kubernetes secret using the provided PEM files:
 
  ::

    kubectl create secret generic tls-group1 \
      --from-file=fullchain.pem=$TUTORIAL_HOME/certs/fullchain.pem \
      --from-file=cacerts.pem=$TUTORIAL_HOME/certs/cacerts.pem \
      --from-file=privkey.pem=$TUTORIAL_HOME/certs/privkey.pem
       
============================
Configure Confluent Platform
============================

You install Confluent Platform components as custom resources (CRs). 

In this tutorial, you will configure Zookeeper, Kafka, and Control Center in a
single file and deploy the components with one ``kubectl apply`` command.

The CR configuration file contains a custom resource specification for each
Confluent Platform component, including replicas, image to use, resource
allocations.

Edit the Confluent Platform CR file: ``$TUTORIAL_HOME/confluent-platform.yaml``

Specifically, note that external accesses to Confluent Platform components are
configured using host-based static routing.

The Kafka section of the file is set as follow for external access:

:: 

  Spec:
    listeners:
      external:
        externalAccess:
          type: staticForHostBasedRouting
          staticForHostBasedRouting:
            domain:                              ----- [1]
            port: 443
        tls:
          enabled: true

* [1] Set this to the value of ``$DOMAIN``, your Kubernetes cluster domain.

* The prefixes are used for external DNS hostnames. In this tutorial, Kafka bootstrap server will use the default prefix, ``kafka``, and the brokers will use the default prefix, ``b``. 

  As Kafka is configured with 3 replicas in this tutorial, the access endpoints of
  Kafka will be:
  
  * kafka.$DOMAIN for the bootstrap server
  * b0.$DOMAIN for the broker #1
  * b1.$DOMAIN for the broker #2
  * b2.$DOMAIN for the broker #3

=========================
Deploy Confluent Platform
=========================

#. Deploy Confluent Platform with the above configuration:

   ::

     kubectl apply -f $TUTORIAL_HOME/confluent-platform.yaml

#. Check that all Confluent Platform resources are deployed:

   ::
   
     kubectl get confluent

#. Get the status of any component. For example, to check Kafka:

   ::
   
     kubectl describe kafka
     
=========================
Deploy Ingress controller
=========================

An Ingress controller is required to access Kafka using the static host-based
routing. In this tutorial, we will use Nginx Ingress controller.

SSL passthrough is the action of passing data through a load balancer to a server without decrypting it. 
In many load balancer use cases, the decryption or SSL termination happens at the load balancer and data is passed along to the endpoint. 
But SSL passthrough keeps the data encrypted as it travels through the load balancer - and this is what Kafka expects.

#. Clone the Nginx Github repo:

   ::
   
     git clone https://github.com/helm/charts/tree/master/stable/nginx-ingress

#. Install the Ngix controller:

   ::
   
     helm upgrade --install nginx-operator stable/nginx-ingress \
       --set controller.publishService.enabled=true \
       --set controller.extraArgs.enable-ssl-passthrough="true"
       
================================
Create a Kafka bootstrap service
================================

When using staticForHostBasedRouting as externalAccess type, the bootstrap
endpoint is not configured to access Kafka. 

If you want to have a bootstrap endpoint to access Kafka instead of using each
broker's endpoint, you need to provide the bootstrap endpoint, create a
DNS record pointing to Ingress controller load balancer's external IP, and
define the ingress rule for it.

Create the Kafka bootstrap service to access Kafka:

::

  kubectl apply -f $TUTORIAL_HOME/kafka-bootstrap-service.yaml

======================
Create Ingress service
======================

Create an Ingress resource that includes a collection of rules that the Ingress
control uses to route the inbound traffic to Kafka:

#. In the resource file, ``ingress-service-hostbased.yaml``, replace ``$DOMAIN`` 
   with the value of your ``$DOMAIN`` and uncomment the ``hosts:`` and ``host:`` settings.

#. Create the Ingress resource:

   ::

     kubectl apply -f $TUTORIAL_HOME/ingress-service-hostbased.yaml

===============
Add DNS records
===============

Create DNS records for Kafka brokers using the ingress controller load balancer externalIP.

#. Retrieve the external IP addresses of Nginx load balancer:

   ::
   
     kubectl get svc
     
#. Add DNS records for the Kafka brokers using the IP address and
   replacing ``$DOMAIN`` with the actual domain name of your Kubernetes cluster.

   In this tutorial, we are using the default prefixes for components and brokers as shown below:
   
   ====================== ===============================================================
   DNS name               IP address
   ====================== ===============================================================
   kafka.$DOMAIN          The ``EXTERNAL-IP`` value of the Nginx load balancer service
   b0.$DOMAIN             The ``EXTERNAL-IP`` value of the Nginx load balancer service
   b1.$DOMAIN             The ``EXTERNAL-IP`` value of the Nginx load balancer service
   b2.$DOMAIN             The ``EXTERNAL-IP`` value of the Nginx load balancer service
   ====================== ===============================================================
  
========
Validate
========

Deploy producer application
^^^^^^^^^^^^^^^^^^^^^^^^^^^

Now that we've got the Confluent Platform set up, let's deploy the producer
client app.

The producer app is packaged and deployed as a pod on Kubernetes. The required
topic is defined as a KafkaTopic custom resource in
``$TUTORIAL_HOME/producer-app-data.yaml``.

In a single configuration file, you do all of the following:

* Provide client credentials.

  Create a Kubernetes secret with the ``kafka.properties`` file and a pre-generated
  keystore and trustore.
  
* Deploy the producer app.

* Create a topic for it to write to.

  The ``$TUTORIAL_HOME/producer-app-data.yaml`` defines the ``elastic-0`` topic as
  follows:

  ::
  
    apiVersion: platform.confluent.io/v1beta1
    kind: KafkaTopic
    metadata:
      name: elastic-0
      namespace: confluent
    spec:
      replicas: 1
      partitionCount: 1
      configs:
        cleanup.policy: "delete"
  
**To deploy the producer application:**

#. Generate an encrypted ``kafka.properties`` file content:

   ::
   
     echo bootstrap.servers=kafka.$DOMAIN:443 \
       security.protocol=SSL \
       ssl.truststore.location=/mnt/truststore.jks \
       ssl.truststore.password=mystorepassword | base64
   
#. Provide the output from the previous step for ``kafka.properties`` in the 
   ``$TUTORIAL_HOME/producer-app-data.yaml`` file:

   ::
   
     apiVersion: v1
     kind: Secret
     metadata:
       name: kafka-client-config
       namespace: confluent
     type: Opaque
     data:
       kafka.properties: # Provide the base64-encoded kafka.properties

#. Deploy the producer app:

   ::
   
     kubectl apply -f $TUTORIAL_HOME/producer-app-data.yaml

Validate in Control Center
^^^^^^^^^^^^^^^^^^^^^^^^^^

Use Control Center to monitor the Confluent Platform, and see the created topic and data.

#. Set up port forwarding to Control Center web UI from local machine:

   ::

     kubectl port-forward controlcenter-0 9021:9021

#. Browse to `Control Center <https://localhost:9021>`__.

#. Check that the ``elastic-0`` topic was created and that messages are being produced to the topic.

=========
Tear Down
=========

Shut down Confluent Platform and the data:

::

  kubectl delete -f $TUTORIAL_HOME/producer-app-data.yaml
  
::

  kubectl delete -f $TUTORIAL_HOME/ingress-service-hostbased.yaml
  
::

  kubectl delete -f $TUTORIAL_HOME/kafka-bootstrap-service.yaml

::

  kubectl delete -f $TUTORIAL_HOME/confluent-platform.yaml
  
::

  kubectl delete secret tls-group1
  
::

  helm delete nginx-operator

::

  helm delete operator
  
