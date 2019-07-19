# Kafka + Flink on Kubernetes

__Requirements:__

docker:
https://docs.docker.com/docker-for-mac/install/  
minikube: 
https://kubernetes.io/docs/tasks/tools/install-minikube/  
kubectl: 
https://kubernetes.io/docs/tasks/tools/install-kubectl/  
helm: 
https://helm.sh/docs/using_helm/#installing-helm  
virtual box:  
https://www.virtualbox.org/wiki/Downloads

# Apache Kafka / Confluent

Confluent 에서 Kafka helm chart 를 제공합니다.  
https://docs.confluent.io/current/installation/installing_cp/cp-helm-charts/docs/index.html

커스텀 설정을 kafka/values.yaml 에 저장하고 실행


    helm install --name kafka-minik --namespace kafka -f values.yaml confluentinc/cp-helm-charts

minikube 에는 persistent storage class 가 이미 설정 되었기 때문에 바로 statefulset 을 구성 가능하지만
퍼블릭 클라우드에 올리거나 bare metal 에 올리실때는 storage class 를 설정을 해주어야합니다.

*참고로 bare metal 에서 local persistent volumes 가 1.14 에서 GA 가 되어 노드에 있는 볼륨 및 디바이스를 사용이 가능하지만
각 pv 마다 따로 볼륨을 설정을 해줘야해서(dynamic provisioning 이 미지원) 매우우우우 번거롭습니다...*


# Apache Flink

Apache Flink 에서 제공되는 Kubernetes setup guide 입니다.  
https://ci.apache.org/projects/flink/flink-docs-stable/ops/deployment/kubernetes.html

Job Cluster
하나의 잡을 실행하기 위한 구성

Session Cluster
여러개의 잡을 실행하기 위한 구성

플링크의 잡이 만약 fail 되게 되었을때 자동으로 다시시작하게 되며 checkpoint/savepoint 를 활용하여서 exactly once 및 data loss 도 방지 가능합니다.

https://ci.apache.org/projects/flink/flink-docs-release-1.8/ops/state/checkpoints.html  
https://ci.apache.org/projects/flink/flink-docs-release-1.8/ops/state/savepoints.html

세션 클러스터로 구성하면 Web GUI 에서 job submit 이 가능해집니다 따라서 여러 잡을 수행하는것도 가능합니다.

Flink web GUI:  

    kubectl port-forward ${flink-jobmanager-pod-name} 8081 -n flink

# Prometheus
Prometheus helm chart  
https://github.com/helm/charts/tree/master/stable/prometheus

아래와 같이 annotation 을 넣어줘야 metrics를 프로메테우스 서버에서 pull 이 가능해집니다.

      annotations:
        prometheus.io/port: "9249"
        prometheus.io/scrape: 'true'

Prometheus web console:

    kubectl port-forward ${prometheus-pod-name} 9090 -n monitoring

# Grafana
Grafana helm chart  
https://github.com/helm/charts/tree/master/stable/grafana

Kafka dashboard:  
https://github.com/confluentinc/cp-helm-charts/blob/master/grafana-dashboard/confluent-open-source-grafana-dashboard.json

Flink dashboard:  
https://grafana.com/grafana/dashboards/10369

Grafana web dashboard:  

    kubectl port-forward ${grafana-pod-name} 3000 -n monitoring

# Kubernetes Web UI(Dashboard)
Kubernetes 웹 대쉬보드:  
https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/

minikube 에서 실행:  

    minikube dashboard

# Kubernetes CLI tools
shell auto completion(쉘에서 자동완성 기능)  
https://kubernetes.io/docs/tasks/tools/install-kubectl/?source=#enabling-shell-autocompletion

kubectx/kubens(k8s context swith 그리고 k8s namespace 체인져)  
https://github.com/ahmetb/kubectx

kube ps1(쉘 프롬프트를 k8s를 사용하기 편하게 꾸며준다)  
https://github.com/jonmosco/kube-ps1

kube aliases(alias 모음들 너무 많아서 외우기가 힘들다)  
https://github.com/ahmetb/kubectl-aliases


# Cheat sheet

```
# Install minikube and start
brew cask install minikube
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
helm repo add confluentinc https://confluentinc.github.io/cp-helm-charts/
helm repo update
cd kafka
helm install --name kafka-minik --namespace kafka -f values.yaml confluentinc/cp-helm-charts

# Install flink
kubectl create namespace flink
kubectl apply -f ./flink

# Install prometheus + grafana
helm install --name prometheus-minik --namespace monitoring stable/prometheus
helm install --name grafana-minik --namespace monitoring stable/grafana

# Install Minio
helm install --name minio-minik --namespace minio --set accessKey=k8sdemo,secretKey=k8sdemo123 stable/minio

# Connect to kafka container
kubectl exec -n kafka -it kafka-minik-cp-kafka-0 --container cp-kafka-broker bash

# Kafka commands
kafka-topics --list --zookeeper kafka-minik-cp-zookeeper:2181
kafka-topics --zookeeper kafka-minik-cp-zookeeper:2181 --create --topic kafka-demo --partitions 6 --replication-factor 1