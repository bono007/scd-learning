# scd

## Minikube

minikube start --cpus=4 --memory=8192

helm upgrade dflow bitnami/spring-cloud-dataflow -f minikube-values.yaml
helm install dflow bitnami/spring-cloud-dataflow -f minikube-values.yaml

export SERVICE_PORT=$(kubectl get --namespace default -o jsonpath="{.spec.ports[0].port}" services dflow-spring-cloud-dataflow-server)
kubectl port-forward --namespace default svc/dflow-spring-cloud-dataflow-server 8080:8080
echo "http://127.0.0.1:${SERVICE_PORT}/dashboard"


skipper:
  jdwp:
    enabled: true


kubectl port-forward dflow-spring-cloud-dataflow-skipper-78d64964cc-gr4wc 5005:5005

## Questions

Why is `initContainer` different than `.container`? 
    - My guess is the former was around prior to moving to more Yaml content properties?
    

Why not just expose the Fabric8 object model as ConfigurationProperties? The big-yaml properties seem to straddle this approach.
Is it for the safety of the extra layer of indirection (aka don't code to Fabric8)
