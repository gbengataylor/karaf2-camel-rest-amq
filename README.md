# Karaf Camel REST AMQ QuickStart

This example shows how to use Camel in a Karaf Container using Blueprint to connect to the AMQ xPaaS message broker on OpenShift.
The Red Hat JBoss AMQ xPaaS product should already be installed and running on your OpenShift installation - see the [documentation](https://docs.openshift.com/enterprise/3.1/using_images/xpaas_images/a_mq.html)

This example will connect to the AMQ message broker and send messages to an AMQ queue once a REST endpoint `/karaf2-camel-rest-amq/generate/order` is hit.


## Pre-requisites

### Download the required AMQ images and templates

Download the images

    oc -n openshift import-image jboss-amq-63:1.3

Download the AMQ template. In this example, the ephemeral AMQ broker is used.

    oc create -n openshift -f https://raw.githubusercontent.com/jboss-openshift/application-templates/ose-v1.4.8/amq/amq63-basic.json

    oc replace -n openshift -f https://raw.githubusercontent.com/jboss-openshift/application-templates/ose-v1.4.8/amq/amq63-basic.json

Admin rights are required to create in the openshift namespace

### Deploy the Ephermeral AMQ broker

Create the following environment variables. Make the appropriate modifications

    export PROJECT=fis

    export OPENSHIFT_BROKER_APPLICATION_NAME=broker

The discovery agent type to use for discovering mesh endpoints needs to be set. 'dns' will use OpenShift's DNS service to resolve endpoints. 'kube' will use Kubernetes REST API to resolve service endpoints.

In this example, 'kube' is used. The service account for the pod must have the 'view' role, which can be added via

    oc policy add-role-to-user view system:serviceaccount:$PROJECT:default

Deploy the AMQ broker

    oc new-app --template=amq63-basic app=${OPENSHIFT_BROKER_APPLICATION_NAME} --param  APPLICATION_NAME=${OPENSHIFT_BROKER_APPLICATION_NAME} --param AMQ_MESH_DISCOVERY_TYPE=kube --param MQ_PROTOCOL=openwire --param MQ_USERNAME=admin --param MQ_PASSWORD=password -l app=${OPENSHIFT_BROKER_APPLICATION_NAME}



### Download the required FIS 2.0 Karaf images

    oc create -n openshift -f https://raw.githubusercontent.com/jboss-fuse/application-templates/fis-2.0.x.redhat-R6/fis-image-streams.json

Admin rights are required to create in the openshift namespace

The  broker exposes a service on the tcp port name ${OPENSHIFT_BROKER_APPLICATION_NAME}-amq-tcp


### Upload the template to OpenShift

    oc create -n openshift -f karaf2-camel-rest-amq-template.yaml

if it already exists

    oc replace -n openshift -f  karaf2-camel-rest-amq-template.yaml


Admin rights are required to create/replace in the openshift namespace

## Running the example in OpenShift using template


Create the following environment variables. Make the appropriate modifications

    export OPENSHIFT_CAMEL_APPLICATION_NAME=fis-karaf-camel-rest-amq-route

    export GIT_REPO_CAMEL_AMQ=https://github.com/gbengataylor/karaf2-camel-rest-amq.git

    export IMAGE_BUILD_VERSION=2.0

Deploy the camel route

    oc new-app --template=s2i-karaf2-camel-rest-amq --param APP_NAME=${OPENSHIFT_CAMEL_APPLICATION_NAME} GIT_REPO=${GIT_REPO_CAMEL_NO_AMQ}  --param SERVICE_NAME=${OPENSHIFT_CAMEL_APPLICATION_NAME} --param ACTIVEMQ_SERVICE_NAME=${OPENSHIFT_BROKER_APPLICATION_NAME}-amq-tcp --param ACTIVEMQ_USERNAME=admin --param ACTIVEMQ_PASSWORD=password --param BUILDER_VERSION=${IMAGE_BUILD_VERSION} --param GIT_REF=master -l app=${OPENSHIFT_CAMEL_APPLICATION_NAME}
