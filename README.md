# Kafka + Flink on Kubernetes Sample

Requirements:

docker:
https://docs.docker.com/docker-for-mac/install/

minikube: 
https://kubernetes.io/docs/tasks/tools/install-minikube/

kubectl: 
https://kubernetes.io/docs/tasks/tools/install-kubectl/

helm: 
https://helm.sh/docs/using_helm/#installing-helm



# Apache Kafka / Confluent

Confluent 에서 Kafka helm chart 를 제공합니다

https://docs.confluent.io/current/installation/installing_cp/cp-helm-charts/docs/index.html

minikube 에 배포하는 방법:

자신의 설정에 맞는 설정을 values.yaml 에 저장하고 실행

helm install --name <kafka-minik> --namespace <kafka> -f values.yaml confluentinc/cp-helm-charts

아무 문제 없이 설치가 되었다면 


# Aapche Flink

Apache Flink 에서 제공되는 Kubernetes setup guide 입니다
https://ci.apache.org/projects/flink/flink-docs-stable/ops/deployment/kubernetes.html

Job Cluster
하나의 잡을 실행하기 위한 구성

Session Cluster
여러개의 잡을 구성하기 위한 구성

플링크의 잡이 만약 fail 되게 되었을때 자동으로 다시시작하게 되며 checkpoint/savepoint 를 활용하여서 exactly once 및 data loss 도 방지 가능합니다

https://ci.apache.org/projects/flink/flink-docs-release-1.8/ops/state/checkpoints.html

https://ci.apache.org/projects/flink/flink-docs-release-1.8/ops/state/savepoints.html



# Prometheus



# Grafana



# Cheat sheet

```
# Install minikube and start
brew cast install minikube
minikube start

# Install kubectl and move to usr/bin
curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/darwin/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl

# Install helm and update repo
brew install kubernetes-helm
helm init --history-max 200
helm repo update

# Install kafka
helm install --name <kafka-minik> --namespace <kafka> -f values.yaml confluentinc/cp-helm-charts

# Install flink


# Install prometheus + grafana

helm install stable/prometheus
helm install stable/grafana

# Install Minio
helm install --name minio-minik --namespace minio --set accessKey=k8sdemo,secretKey=k8sdemo123 stable/minio