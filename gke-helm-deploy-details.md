




1. modify SCDF/SCDK code and put modified libs in docker build dirs

  cd /Users/cbono/indeed/spring.io/scd/spring-cloud-deployer-kubernetes
  ./mvnw clean install -DskipTests

  cd /Users/cbono/indeed/spring.io/scd/spring-cloud-dataflow
  ./mvnw clean install -DskipTests

  cp spring-cloud-dataflow-server/target/spring-cloud-dataflow-server-2.10.0-SNAPSHOT.jar /Users/cbono/indeed/spring.io/scd/bitnami-docker-spring-cloud-dataflow/2/debian-10/rootfs/opt/bitnami/spring-cloud-dataflow/

  cd /Users/cbono/indeed/spring.io/scd/bitnami-docker-spring-cloud-dataflow/2/debian-10/rootfs/opt/bitnami/spring-cloud-dataflow

  ln -s spring-cloud-dataflow-server-2.10.0-SNAPSHOT.jar spring-cloud-dataflow/spring-cloud-dataflow.jar

  cd /Users/cbono/indeed/spring.io/scd/spring-cloud-skipper
  ./mvnw clean install -DskipTests

  cp spring-cloud-skipper-server/target/spring-cloud-skipper-server-2.9.0-SNAPSHOT.jar /Users/cbono/indeed/spring.io/scd/bitnami-docker-spring-cloud-skipper/2/debian-10/rootfs/opt/bitnami/spring-cloud-skipper/

  cd /Users/cbono/indeed/spring.io/scd/bitnami-docker-spring-cloud-skipper/2/debian-10/rootfs/opt/bitnami/spring-cloud-skipper

  ln -s spring-cloud-skipper-server-2.9.0-SNAPSHOT.jar spring-cloud-skipper.jar

2. Build docker containers

  cd /Users/cbono/indeed/spring.io/scd/bitnami-docker-spring-cloud-dataflow/2/debian-10

  docker build -t cbono007/spring-cloud-dataflow:cv10 -f Dockerfile .

  docker tag cbono007/spring-cloud-dataflow:cv10 us-central1-docker.pkg.dev/bono007-sandbox/bono007-scdf/spring-cloud-dataflow:cv10

  docker push us-central1-docker.pkg.dev/bono007-sandbox/bono007-scdf/spring-cloud-dataflow:cv10


  cd /Users/cbono/indeed/spring.io/scd/bitnami-docker-spring-cloud-skipper/2/debian-10

  docker build -t cbono007/spring-cloud-skipper:cv10 -f Dockerfile .

  docker tag cbono007/spring-cloud-skipper:cv10 us-central1-docker.pkg.dev/bono007-sandbox/bono007-scdf/spring-cloud-skipper:cv10

  docker push us-central1-docker.pkg.dev/bono007-sandbox/bono007-scdf/spring-cloud-skipper:cv10


3. Helm install w/ custom image coordinates

  cd /Users/cbono/indeed/spring.io/scd/
  helm install dflow11 -f gke-values.yaml bitnami/spring-cloud-dataflow




4. Deploy sample apps - see the message in the server app logs.

  spring.cloud.deployer.kubernetes.limit.cpu=500m
  spring.cloud.deployer.kubernetes.limits.memory=600m
  spring.cloud.deployer.kubernetes.requests.cpu=500m
  spring.cloud.deployer.kubernetes.requests.memory=600m








TIP: sudo lsof -i -P | grep LISTEN

./mvnw clean install spring-boot:build-image

docker tag docker.io/library/usage-detail-sender-kafka:0.0.1-SNAPSHOT us-central1-docker.pkg.dev/bono007-sandbox/bono007-scdf/usage-detail-sender:v3

docker push us-central1-docker.pkg.dev/bono007-sandbox/bono007-scdf/usage-detail-sender:v3
