Deploy Confluent Platform with RBAC authorization
=================================================

In this workflow scenario, you'll set up secure Confluent Platform clusters with
SASL PLAIN authentication, role-based access control (RBAC) authorization, and
inter-component TLS.

Before you begin this tutorial:

* `Set up the prerequisites <https://github.com/confluentinc/operator-earlyaccess#pre-requisites>`__.

* `Create the namespace for the tutorials <https://github.com/confluentinc/operator-earlyaccess#set-up-the-kubernetes-cluster>`__.

* `Configure the Early Access credentials <https://github.com/confluentinc/operator-earlyaccess#configure-early-access-credentials>`__.

* `Clone the tutorial repo <https://github.com/confluentinc/operator-earlyaccess#download-confluent-operator-tutorial-package>`__.

To complete this scenario, you'll follow these steps:

#. Set the current tutorial directory.

#. Deploy Confluent Operator.

#. Deploy OpenLDAP.

#. Deploy configuration secrets.

#. Deploy Confluent Platform.

#. Configure a role binding.

#. Validate.

#. Tear down Confluent Platform.

==================================
Set the current tutorial directory
==================================

Set the tutorial directory for this tutorial under the directory you downloaded
the tutorial files:

::
   
  export TUTORIAL_HOME=<Tutorial directory>/cp-rbac-deploy
  
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

===============
Deploy OpenLDAP
===============

This repo includes a Helm chart for `OpenLdap
<https://github.com/osixia/docker-openldap>`__. The chart ``values.yaml``
includes the set of principal definitions that Confluent Platform needs for
RBAC.

#. Deploy OpenLdap

   ::

     helm upgrade --install -f $TUTORIAL_HOME/openldap/ldaps-rbac.yaml test-ldap ./openldap --namespace confluent


#. Validate that OpenLDAP is running:  
   
   ::

     kubectl get pods

#. Log in to the LDAP pod:

   ::

     kubectl exec -it ldap-0 -- bash

#. Run the LDAP search command:

   ::

     ldapsearch -LLL -x -H ldap://ldap.confluent.svc.cluster.local:389 -b 'dc=test,dc=com' -D "cn=mds,dc=test,dc=com" -w 'Developer!'

#. Exit out of the LDAP pod:

   ::
   
     exit 
     
============================
Deploy configuration secrets
============================

You'll use Kubernetes secrets to provide credential configurations.

With Kubernetes secrets, credential management (defining, configuring, updating)
can be done outside of the Confluent Operator. You define the configuration
secret, and then tell Confluent Operator where to find the configuration.

In this tutorial, you will deploy a secure Zookeeper, Kafka and Control Center,
and the rest of Confluent Platform components as shown below:

.. figure:: ../images/confluent-platform-security-deployment.png
   :width: 200px
   
To support the above deployment scenario, you need to provide the following
credentials:

* Root Certificate Authority to auto-generate certificates

* Authentication credentials for Zookeeper, Kafka, and Control Center

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
       -subj "/C=US/ST=CA/L=MountainView/O=Confluent/OU=Opeator/CN=TestCA"

#. Create a Kuebernetes secret for inter-component TLS:

   ::

     kubectl create secret tls ca-pair-sslcerts \
       --cert=$TUTORIAL_HOME/ca.pem \
       --key=$TUTORIAL_HOME/ca-key.pem
  
Provide authentication credentials
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

#. Create a Kubernetes secret object for Zookeeper, Kafka, and Control Center. 

   This secret object contains file based properties. These files are in the
   format that each respective Confluent component requires for authentication
   credentials.

   ::
   
     kubectl create secret generic credential \
      --from-file=plain-users.json=$TUTORIAL_HOME/creds-kafka-sasl-users.json \
      --from-file=digest-users.json=$TUTORIAL_HOME/creds-zookeeper-sasl-digest-users.json \
      --from-file=digest.txt=$TUTORIAL_HOME/creds-kafka-zookeeper-credentials.txt \
      --from-file=plain.txt=$TUTORIAL_HOME/creds-client-kafka-sasl-user.txt \
      --from-file=basic.txt=$TUTORIAL_HOME/creds-control-center-users.txt \
      --from-file=ldap.txt=$TUTORIAL_HOME/ldap.txt

   In this tutorial, we use one credential for authenticating all client and
   server communication to Kafka brokers. In production scenarios, you'll want
   to specify different credentials for each of them.

#. Create Kubernetes secret objects for MDS:

   ::
   
     kubectl create secret generic mds-token \
       --from-file=mdsPublicKey.pem=$TUTORIAL_HOME/mds-publickey.txt \
       --from-file=mdsTokenKeyPair.pem=$TUTORIAL_HOME/mds-tokenkeypair.txt
   
   ::
   
     kubectl create secret generic mds-client \
       --from-file=bearer.txt=$TUTORIAL_HOME/bearer.txt

========================================
Review Confluent Platform configurations
========================================

You install Confluent Platform components as custom resources (CRs).

The Confluent Platform components are configured in one file for secure
authentication and encryption for:
``$TUTORIAL_HOME/confluent-platform-rbac-secure.yaml``

Let's take a look at how these components are configured.

Kafka RBAC configuration
^^^^^^^^^^^^^^^^^^^^^^^^

::

  apiVersion: platform.confluent.io/v1beta1
  kind: Kafka
  metadata:
    name: kafka
    namespace: confluent
  spec:
    authorization:
      type: rbac                                                  --- [1]
      superUsers:                                                 --- [2]
      - User:kafka
      - User:ANONYMOUS
    services:
      restProxy:                                                  --- [3]
        enabled: true
        mds:
          authentication:
            type: bearer
            bearer:
              secretRef: mds-client                               --- [4]
      mds:                                                        --- [5]
        tls:
          enabled: true                                           --- [6]
        tokenKeyPair:
          secretRef: mds-token
        ldap:
          address: ldap://ldap.confluent.svc.cluster.local:389    --- [7]
          authentication:
            type: simple
            simple:
              secretRef: credential                               --- [8]
          configurations:                                         --- [9]
            groupNameAttribute: cn
            groupObjectClass: group
            groupMemberAttribute: member
            groupMemberAttributePattern: CN=(.*),DC=test,DC=com
            groupSearchBase: dc=test,dc=com
            userNameAttribute: cn
            userMemberOfAttributePattern: CN=(.*),DC=test,DC=com
            userObjectClass: organizationalRole
            userSearchBase: dc=test,dc=com

* [1] Enable the RBAC authorization.

* [2] MDS super user. This user bypasses authorization and is authenticated through LDAP.

* [3] REST Proxy is required for RBAC.

* [4] The Kubernetes secret for the MDS key token pair created above.

* [5] MDS is required for RBAC.

* [6] TLS is enabled for MDS in this tutorial.

* [7] URL for the LDAP used in this tutorial, OpenLdap.

* [8] The Kubernetes secret for LDAP credential.

* [9] LDAP settings for this tutorial.

Control Center RBAC configuration
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

::

  ---
  apiVersion: platform.confluent.io/v1beta1
  kind: ControlCenter
  metadata:
    name: controlcenter
    namespace: confluent
  spec:
    authorization:
      type: rbac                                                  --- [1]
    dependencies:
      mds:                                                        --- [2]
        endpoint: https://kafka.confluent.svc.cluster.local:8090  --- [3]
        tokenKeyPair:                
          secretRef: mds-token                                    --- [4]
        authentication:
          type: bearer
          bearer:
            secretRef: mds-client                                 --- [5]
        tls:
          enabled: true                                           --- [6]

* [1] The RBAC authorization is enabled.

* [2] MDS dependency is required for RBAC.

* [3] The MDS endpoint. This is set to the internal endpoint to MDS in this tutorial.

* [4] The Kubernetes secret for MDS token key pair.

* [5] The Kubernetes secret for MDS credential.

* [6] TLS is enabled for MDS in this tutorial. 

=========================
Deploy Confluent Platform
=========================

#. Deploy Confluent Platform with the above configuration:

   ::

     kubectl apply -f $TUTORIAL_HOME/confluent-platform-rbac-secure.yaml

#. Check that all Confluent Platform resources are deployed:

   ::
   
     kubectl get pods
     
#. In the output from the previous step, note that the ``READY`` column for ``controlcenter-0`` pod is ``0/1``. The Control Center service cannot be ready until RBAC is configure.

========================
Configure a role binding
========================

#. Set up port forwarding to the MDS server:

   ::
   
     kubectl port-forward kafka-0 8090:8090

#. Add the following in your local ``/etc/hosts`` file. This is a workaround for the self-signed certificate we are using in this tutorial.

   ::
   
     127.0.0.1	kafka.confluent.svc.cluster.local

#. Log into MDS with the ``kafka`` user and the ``kafka-secret`` password:

   ::
   
     confluent login --url https://kafka.confluent.svc.cluster.local:8090 \
       --ca-cert-path $TUTORIAL_HOME/ca.pem

#. Get Kafka cluster id:

   ::
   
     curl -ik https://kafka.confluent.svc.cluster.local:8090/v1/metadata/id 
     
#. Take the id value in the above output and save it as an environment variable:

   ::
   
     export KAFKA_ID=<Kafka cluster id>

#. Create Control Center Role Binding for the ``c3`` user:

   ::
   
     confluent iam rolebinding create \
       --principal User:c3 \
       --role SystemAdmin \
       --kafka-cluster-id $KAFKA_ID
       
#. Control Center will restart in 50 seconds. Run the following command to verify that Control Center is up and ready:

   ::
   
     kubectl get pods
     
   The ``READY`` column for ``controlcenter-0`` should have ``1/1``.

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

#. Browse to Control Center. You will log in as the ``c3`` user as set in ``$TUTORIAL_HOME/creds-control-center-users.txt``:

   ::
   
     https://localhost:9021

The ``c3`` user has the ``SystemAdmin`` role granted and will have access to the
cluster and broker information.

=========
Tear down
=========

::

  kubectl delete -f $TUTORIAL_HOME/confluent-platform-rbac-secure.yaml

::

  kubectl delete secret mds-token
  
::

  kubectl delete secret mds-client

::

  kubectl delete secret credential

::

  kubectl delete secret ca-pair-sslcerts

::

  helm delete operator
  
