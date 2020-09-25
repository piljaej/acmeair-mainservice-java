
## Acme Air Main Service - Java/Liberty

An implementation of the Acme Air Main Service for Java/Liberty. The main service primarily consists of the presentation layer (web pages) that interact with the other services.

This implementation can support running on a variety of runtime platforms including standalone bare metal system, Virtual Machines, docker containers, IBM Cloud, IBM Cloud Private, and other Kubernetes environments.

## Build Instructions
* Instructions for [setting up and building the codebase](Build_Instructions.md)

## JMeter Instructions
* Instructions for [driving load to the app after it has been deployed](https://github.com/blueperf/acmeair-mainservice-java/tree/master/jmeter-files)

## Full Benchmark Installation Instructions on various docker enviornments.
![alt text](https://github.com/blueperf/acmeair-mainservice-java/blob/master/images/AcmeairMS.png "AcmeairMS Java")

## Prerequisites
All of these examples assume you have the acmeair-mainservice-java, acmeair-authservice-java, acmeair-bookingservice-java, acmeair-customerservice-java, and acmeair-flightservice-java directories, (and possibly others) on your docker machine in the same directory. It also assumes all applications have been built with maven.


## Docker Instructions

Prereq: [Install Docker, docker-compose, and start Docker daemon on your local machine](https://docs.docker.com/installation/)

1. cd acmeair-mainservice-java
2. Create docker network
 * docker network create --driver bridge my-net
3. Build/Start Containers. This will build all the micro-services, mongo db instances, and an nginx proxy.
    * docker-compose build --pull
    * NETWORK=my-net docker-compose up

4. Go to http://docker_machine_ip/acmeair
5. Go to the Configuration Page and Load the Database

## Openshift Instructions
This doc assumes that you are using podman
* Openshift is installed and configured, and internal image registry is available and its default-route is exposed.
* The docker or podman env is logged into the openshift image repository
  Example:
  * oc login https://api.wasocp44k.fake-link.com:6443 -u ocpadmin -p fake-password
  * docker login -u ocpadmin -p $(oc whoami -t) default-route-openshift-image-registry.apps.wasocp44k.fake-link.com
  or
  * podman login -u ocpadmin -p $(oc whoami -t) --tls-verify=false default-route-openshift-image-registry.apps.wasocp44k.fake-link.com

1. Create new project: oc new-project acmeair
2. Determine default-route (The route you logged into above)
   * Example: default-route-openshift-image-registry.apps.wasocp44k.fake-link.com
3. Determine internal-route.
   * Example (usually): image-registry.openshift-image-registry.svc:5000
2. Determine host for the route. 
   * Example: acmeair.apps.wasocp44k.fake-link.com (The part after acmeair. should match other routes for you cluster) 
3. Run ./buildAndDeployToOpenshift.sh default-route/project_name internal-route/project_name host [podman]
   * Example: ./buildAndDeployToOpenshift.sh default-route-openshift-image-registry.apps.wasocp44k.fake-link.com/acmeair image-registry.openshift-image-registry.svc:5000/acmeair acmeair.apps.wasocp44k.fake-link.com
   Add podman as the last argument if using podman.

### Addon: Openshift Application Metrics (Optional)
1. Create new project: oc new-project app-metrics
2. On the Openshift console under "Operators", click "Operator Hub" and install the following operators to your individual project.
    * Install "Grafana Operator"
    * Install "Prometheus Operator"
3. Run ./setupOpenshiftMetrics.sh project_name app_metrics_project_name
    * Example: ./setupOpenshiftMetrics.sh acmeair app-metrics
4. Visit the Grafana URL in the "Networking" > "Routes" section and click to open the pre-loaded dashboard.

### Addon: Openshift Application Tracing (Optional)
1. On the Openshift console, navigate to your acmeair project
2. The following environment variables are already setup in each acmeair deployment for your ease of use.
    * JAEGER_AGENT_HOST - jaeger-all-in-one-inmemory-agent
    * JAEGER_AGENT_PORT - 6832
    * JAEGER_ENDPOINT - http://jaeger-all-in-one-inmemory-collector:14268/api/traces
3. To get more data, you can optionally enable access logs and tracing for each service by updating the following env variables.
```
    - name: ACCESS_LOGGING_ENABLED
      value: 'true'
    - name: TRACE_SPEC
      value: 'com.acmeair*=all'
```
4. Under "Operators", click "Operator Hub" and install the "Community Jaeger Operator" to your individual project.
5. Click "Installed Operators" and for the newly installed operator create a new Jaeger instance.
6. Visit the Jaeger URL in the "Networking" > "Routes" section and load test the application to see some traces.

## Minikube Instructions

* Prereq: [Install Docker, docker-compose, and start Docker daemon on your local machine](https://docs.docker.com/installation/)
* Prereq: [Install and configure Minikube, and install kubectl](https://github.com/kubernetes/minikube/)

1. minikube docker-env
2. eval $(minikube docker-env)
3. minikube addons enable ingress
4. cd acmeair-mainservice-java/scripts
5. Build and Deploy Services
  ./buildAndDeployToMinikube.sh
6. Wait a couple minutes and go to http://kubernetes_ip/acmeair
7. Go to the Configuration Page and Load the Database

## IBM Cloud Private Instructions
This doc assumes that
* ICP is installed and configured
* The docker env is logged into the ICP docker repo
  * docker login mycluster.icp:8500

* kubectl and helm are attached the ICP cluster

* You are running ICP as admin

* maven is installed and set up to build with a full SDK.

1. Build and push the apps
   * `cd acmeair-mainservice-java/scripts`
   * `./buildAndPushtoICP.sh`
2. Deploy to ICP using one of the following options: 
   * Using ibm-websphere-liberty helm chart
      * `./deployChartToICP.sh`
   * Using loose deployment manifests
     * `./deployToICP.sh`
3. Wait a couple minutes and go to http://proxy_ip/acmeair
4. Go to the Configuration Page and Load the Database
5. Cleanup
   * Helm chart
      * `./deleteChartRelease.sh`
   * Loose deployment manifests
      * `./deleteKubeObjects.sh`

## Istio Instructions

There is also a sample [manifest file](./manifests-istio/deploy-acmeair-istio.yaml) for deployment of the Acmeair application in a Istio service mesh, with all services and deployments required to run the application, including the gateway and virtual services definition. The required Mongo databases are also deployed as pods/services. You need to define the docker images to be used and inject the sidecars, either automatically or manually (with kubectl kube-inject command).  
