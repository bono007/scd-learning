# scd

## Minikube
```
minikube start --cpus=4 --memory=8192

helm upgrade dflow bitnami/spring-cloud-dataflow -f minikube-values.yaml
helm install dflow bitnami/spring-cloud-dataflow -f minikube-values.yaml

export SERVICE_PORT=$(kubectl get --namespace default -o jsonpath="{.spec.ports[0].port}" services dflow-spring-cloud-dataflow-server)
kubectl port-forward --namespace default svc/dflow-spring-cloud-dataflow-server 8080:8080
echo "http://127.0.0.1:${SERVICE_PORT}/dashboard"
```

## Questions

Q1: Why is `initContainer` different than `.container`? My guess is the former was around prior to moving to more Yaml content properties?

Q2: Why not just expose the Fabric8 object model as ConfigurationProperties? The big-yaml properties seem to straddle this approach. Is it for the safety of the extra layer of indirection (aka don't code to Fabric8)

Q3: When adding a set of properties, how de we decide on "how much" of the Fabric properties to expose to user? An example is adding the container security.
The ask is to add `allowPrivilegeEscalation` and `readOnlyRootFilesystem`. However, the container `SecurityContext` in Fabric8 [exposes more](https://www.javadoc.io/doc/io.fabric8/kubernetes-model-core/latest/io/fabric8/kubernetes/api/model/SecurityContext.html). 
Also, you can see in the current support for PodSecurityContext we also leave out several of the available attributes on the [Fabric8 model](https://www.javadoc.io/doc/io.fabric8/kubernetes-model-core/latest/io/fabric8/kubernetes/api/model/PodSecurityContext.html).

Q4: How does the version of Kubernetes client come into SCDK?

Q5: What is our team involvement in [spring-cloud-kubernetes](https://github.com/spring-cloud/spring-cloud-kubernetes) ? 

Q: Why does `spring-cloud-dataflow-autoconfigure` include `kubernetes-client` when it gets brought in via its other include to `spring-cloud-dataflow-platform-kubernetes` ?
      TODO: VERIFY I think because it uses SCK to get its config maps and SCK probably requires a specific client on classpath to enable its auto-config 
        
Q: Why Fabric8 k8s client over k8s client? 

Q: How does SCDF use SCK ? 
    for secrets property sources


Q: Why is `AbstractIntegrationTests, AbstractTaskLauncherIntegrationTests, AbstractAppDeployerIntegrationTests` still around - 
looks like they are dead code w/ no concrete subclasses. The concrete subclasses all derive from the Junit5 ABT replacements.
  

### Maven questions
Q1: What is <dependencyManagement> and how is it different than <dependencies> ?
Q2: When pom.xml has a parent, what all does it "inherit" from that parent?


### Integration tests
```
gcloud container --project bono007-sandbox clusters create "spring-test" --zone "us-central1-b" --machine-type "n1-highcpu-2" --scopes "https://www.googleapis.com/auth/compute","https://www.googleapis.com/auth/devstorage.read_only","https://www.googleapis.com/auth/logging.write" --network "default"
gcloud config set container/cluster spring-test
gcloud config set compute/zone us-central1-b
gcloud container clusters get-credentials spring-test
kubectl version
```

### SCDF Sandbox
```
gcloud container clusters delete scdf

gcloud container clusters create scdf \
  --project bono007-sandbox \
  --zone "us-central1-b" \
  --network default \
  --machine-type e2-standard-4

gcloud config set container/cluster scdf
gcloud config set compute/zone us-central1-b
gcloud container clusters get-credentials scdf
gcloud auth application-default login
kubectl version
helm install my-scdf --set kafka.enabled=true,rabbitmq.enabled=false  bitnami/spring-cloud-dataflow

gcloud artifacts repositories create bono007-scdf --repository-format=docker \
--location=us-central1 --description="My SCDF repository"

gcloud artifacts repositories list

gcloud auth configure-docker us-central1-docker.pkg.dev
OR
gcloud auth print-access-token | docker login -u oauth2accesstoken \
    --password-stdin https://us-central1-docker.pkg.dev

cd usage-detail-sender-kafka && \
./mvnw clean install spring-boot:build-image && \
docker tag docker.io/library/usage-detail-sender-kafka:0.0.1-SNAPSHOT us-central1-docker.pkg.dev/bono007-sandbox/bono007-scdf/usage-detail-sender:v3 && \
docker push us-central1-docker.pkg.dev/bono007-sandbox/bono007-scdf/usage-detail-sender:v3 && \
cd ../usage-cost-processor-kafka && \
./mvnw clean install spring-boot:build-image && \
docker tag docker.io/library/usage-cost-processor-kafka:0.0.1-SNAPSHOT us-central1-docker.pkg.dev/bono007-sandbox/bono007-scdf/usage-cost-processor:v3 && \
docker push us-central1-docker.pkg.dev/bono007-sandbox/bono007-scdf/usage-cost-processor:v3 && \
cd ../usage-cost-logger-kafka && \
./mvnw clean install spring-boot:build-image && \
docker tag docker.io/library/usage-cost-logger-kafka:0.0.1-SNAPSHOT us-central1-docker.pkg.dev/bono007-sandbox/bono007-scdf/usage-cost-logger:v3 && \
docker push us-central1-docker.pkg.dev/bono007-sandbox/bono007-scdf/usage-cost-logger:v3 && \
cd ..
```

Deployment property:
```
kubernetes.createLoadBalancer
```
