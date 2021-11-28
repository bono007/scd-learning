
## Mission
Make a minor change to the k8s deployer codebase and see my changes deployed in GKE.

### Legend
* GCP  = Google Cloud Platform
* GKE  = Google Kubernetes Engine
* SCDF = Spring Cloud Dataflow
* SCS  = Spring Cloud Skipper 

### Pre-requisites
* GCP account created
* GKE cluster created
* SCDF + SCS running on GKE cluster (installed w/ Helm)

### Code change
I modified the `KubernetesAppDeployer` with a `logger.warn("*** BONO WAS HERE")` call.

### Deployment

I leveraged Helm for the install/deployment of the code change to GKE. 

#### Default install
The Helm chart points the k8s "server" and "skipper" container images to the Bitnami Docker registry/repos:
* [docker.io/bitnami/spring-cloud-dataflow:2.9.1-debian-10-r0](https://hub.docker.com/r/bitnami/spring-cloud-dataflow)
* [docker.io/bitnami/spring-cloud-skipper:2.8.0-debian-10-r15](https://hub.docker.com/r/bitnami/spring-cloud-skipper)

The Bitnami docker containers download libs from Bitnami (java, yq, gosu, scdf):
* https://downloads.bitnami.com/files/stacksmith/spring-cloud-dataflow-2.9.1-0-linux-amd64-debian-10.tar.gz
* https://downloads.bitnami.com/files/stacksmith/spring-cloud-skipper-2.8.0-0-linux-amd64-debian-10.tar.gz

#### Modified install
* I cloned the Bitnami docker container repos and modified them to not download the scdf/scs jars (Dockerfile).
* I put my modified scdf/scs jars in the location it is normally downloaded to:
    * /bitnami-docker-spring-cloud-dataflow/2/debian-10/rootfs/opt/bitnami/spring-cloud-dataflow
    * /bitnami-docker-spring-cloud-dataflow/2/debian-10/rootfs/opt/bitnami/spring-cloud-skipper
* I rebuilt the modified Bitnami docker containers and pushed to my GKE artifact registry.
* I ran helm w/ custom values to point the `(server|skipper).image.regsitry/repository/tag` to the modified containers in my GKE artifact registry.

#### Results
At this point SCDF/SCS are up and running w/ my code changes. I then deployed the sample stream apps (`usage-detail|usage-cost-processor|usage-cost-logger`) 
and saw the `"*** BONO WAS HERE"` message in the SCS application log. 

> :information_source: The k8s deployer code runs in SCS and the modification to the SCDF container could have been omitted. 

### Questions

#### Recommended development approach
I went w/ the Helm approach in order to learn more about that flow and how it relates to Bitnami, etc. I learned a ton, but 
the above process required jumping through many hoops to get my changes deployed. I am 99.99% sure the above process
is not what the team (we) do when developing on the k8s part of the codebase.

Since the k8s code deployment unit is in terms of containers, what is the most streamlined way to make a change to the k8s deployer 
(and really any SCDF/SCS lib) and get it updated in GKE? For Spring Boot stream or task applications we can use the Spring Boot maven plugin
to generate an OCI container image that can be used. Does this work for SCD/SCS install itself? 

What is the recommended approach when developing on the k8s part of the codebase? I know that the k8s deployer requires its 
test to run against GKE as it verifies load balancer things and Minikube does not suit this well. Does everyone run against 
Minkube first and then do a "final" test against GKE? Or does everyone run against GKE and have a streamlined approach to 
updating the bits for a quick developer roundtrip?

#### Templates/Descriptors
1. In the SCDF repo why are there duplicate docker-compose and kubernetes descriptors? The templates look identical to the others. 
    * spring-cloud-dataflow/src/docker-compose/ and spring-cloud-dataflow/src/templates/docker-compose/
    * spring-cloud-dataflow/src/kubernetes and spring-cloud-dataflow/src/templates/kubernetes
    
2. The Bitnami Helm chart has descriptors that are almost identical to the k8s descriptors in the SCDF repo. How are they related?
What is the process for updating the Helm chart? Do we own the Helm chart? 

3. On the same vein as the Helm chart questions, what is the process for updating and who owns the Bitnami docker containers? 

4. How do the Carvel templates fit into the above landscape?
