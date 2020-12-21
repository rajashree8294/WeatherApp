# Prereq helm charts for running applications on kubernetes cluster

This helm chart is using StatefulSet of Kubernetes-Kafka.

## Team Members:
1. Rajashree Joshi
1. Kinnar Kansara

## Reuirements
1. Kubernetes 1.3 with alpha APIs enabled and support for storage classes
1. PV support on underlying infrastructure
1. Requires at least v2.0.0-beta.1 version of helm to support dependency management with requirements.yaml

## Helmfile

https://github.com/roboll/helmfile

### Deps
For downloading chart dependencies, use 
```
helmfile deps
```

### Sync
To sync your cluster state as described in your `helmfile`. **Execute this when deploying services for the first time.**
```
helmfile sync
```

### Apply
Begins by executing `diff`. If `diff` finds that there is any changes, sync is executed
```
helmfile apply
```

### destroy
deletes and purges all the releases defined in the manifests
```
helmfile destroy
```

## Repos and Helm Charts used
1. Helm Stable
    
   https://charts.helm.sh/stable (https://github.com/helm/charts/tree/master/stable)

1. Helm Incubator

   http://storage.googleapis.com/kubernetes-charts-incubator (https://github.com/helm/charts/tree/master/incubator)

1. Helm ElasticSearch with Kibana

    https://helm.elastic.co (https://github.com/elastic/helm-charts)

1. Fluentd Elasticsearch

    https://kokuwaio.github.io/helm-charts

1. Prometheus, Alertmanager

    https://prometheus-community.github.io/helm-charts

1. Grafana

    https://grafana.github.io/helm-charts


## Grafana Dashboard

Run Server
```
export POD_NAME=$(kubectl get pods --namespace default -l "app.kubernet.io/name=grafana,app.kubernetes.io/instance=grafana" -o jsonpath="{.items[0].metadata.name}")

kubectl --namespace default port-forward $POD_NAME 3000
```

For getting password for `admin` (default) user,
```
kubectl get secret --namespace default grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
```

### Kafka Dashboards
After opening http://localhost:3000 in browser,

1. Add your first data source. Select Prometheus as data source and select browser in http access. Save & Test.
1. Select Dashboard from left menu and then import. Grafana Dashboard ID: 7589, name: Kafka Exporter Overview. Details: https://grafana.com/dashboards/7589

** Other good dashboards are with ids 721 and 762 **

### References
- https://github.com/helm/charts/tree/master/incubator/kafka
- https://medium.com/better-programming/how-to-run-highly-available-kafka-on-kubernetes-a1824db8a3e2

### ** Below information and usage are deprecated **


## StatefulSet Details
- https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/

## Incubator (Apache Kafka Helm Chart)
- https://github.com/helm/charts/tree/master/incubator/kafka

## Chart Details
This chart will do the following:
- Implement a dynamically scalable kafka cluster using Kubernetes StatefulSets
- Implement a dynamically scalable zookeeper cluster as another Kubernetes StatefulSet required for the Kafka cluster above

## Installing the chart in default namespace
```
helm repo add incubator http://storage.googleapis.com/kubernetes-charts-incubator --force-update

helm install kafka incubator/kafka -f helm/values.yaml
```

## Installing the chart with new namespace
To install the chart with the release name `kafka` in the `kafka` namespace:
```
helm repo add incubator http://storage.googleapis.com/kubernetes-charts-incubator --force-update

kubectl create ns kafka

helm install kafka --namespace kafka incubator/kafka -f helm/values.yaml
```

This will spin up the pods with mentioned configuration in the file `values.yaml`. 
Default configuration is like below:
- 3 Zookeeper pods
- 3 Kafka brokers

And two kafka topics would be created with names `watch` and `weather` with below configuration:
- 3 Partitions
- 2 Replication factor


## Verifying the installation
We can use the Kafka client image pulled from docker hub to test if the setup was successful and kafka is running properly.
```
kubectl apply -f kafka_client.yaml
```

Create a producer that will publish messages to the topic.
```
kubectl  exec -ti testclient -- ./bin/kafka-console-producer.sh --broker-list kafka:9092 --topic watch
```

Open consumer session which can consume the messages sent by producer.
```
kubectl exec -ti testclient -- ./bin/kafka-console-consumer.sh --bootstrap-server kafka:9092 --topic watch
```



