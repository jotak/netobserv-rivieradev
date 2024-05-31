# NetObserv au RivieraDev 2024!

Ce repo contient les instructions et resources utiles au déroulement du [Deep Dive NetObserv](https://www.rivieradev.fr/session/226).

## Diagramme NetObserv

https://docs.openshift.com/container-platform/4.15/observability/network_observability/understanding-network-observability-operator.html

### Without Kafka

![Without Kafka](./images/diag-no-kafka.png)

### With Kafka

![Without Kafka](./images/diag-kafka.png)


## Installation

### OpenShift / OperatorHub

Vous pouvez utiliser la console OpenShift pour installer l'opérateur NetObserv (ou Network Observability) ainsi que Loki.

Pour commencer, le plus simple sera de faire sans Loki:

```bash
# Configurer NetObserv (resource FlowCollector)
kubectl apply -f https://raw.githubusercontent.com/jotak/netobserv-rivieradev/main/flowcollector-no-loki.yaml
```

Si vous utilisez l'opérateur community, vous devrez activer le "User workload monitoring" d'OpenShift, de façon à activer Prometheus pour tous les workloads:

```bash
# Activer le "User workload monitoring" (c'est juste un flag dans une config map)
kubectl apply -f https://raw.githubusercontent.com/jotak/netobserv-rivieradev/main/openshift-user-workload-monitoring.yaml
```

#### Loki Operator

C'est la partie pas facile de l'installation. Commençons par un lien vers la doc: https://docs.openshift.com/container-platform/4.15/observability/network_observability/installing-operators.html#network-observability-loki-installation_network_observability

La doc mentionne principalement comment configurer Loki avec AWS / S3. Cependant d'autres options sont possibles: https://docs.openshift.com/container-platform/4.15/observability/logging/log_storage/installing-log-storage.html#logging-loki-storage_installing-log-storage.

Il est aussi possible d'utiliser Grafana Cloud.

Une fois la doc suivie & le Secret créé dans le namespace netobserv, reste à configurer un LokiStack et le FlowCollector:

```yaml
# Configurer LokiStack (à adapter éventuellement: référence du secret, size, storageClassName...)
kubectl apply -n netobserv -f https://raw.githubusercontent.com/jotak/netobserv-rivieradev/main/lokistack.yaml

# Configurer NetObserv (resource FlowCollector)
kubectl apply -f https://raw.githubusercontent.com/jotak/netobserv-rivieradev/main/flowcollector-loki-operator.yaml
```


### KIND

```bash
# Cloner le repo
git clone git@github.com:netobserv/network-observability-operator.git
cd network-observability-operator

# Démarrer KIND (pour podman, il faut lancer en root, cf plus bas)
kind create cluster
kubectl config set-context --current --namespace=netobserv

# Déployer netobserv
USER=netobserv make deploy-kind

# Déployer Grafana et Loki
make deploy-grafana deploy-loki

# Configurer NetObserv (resource FlowCollector)
kubectl apply -f https://raw.githubusercontent.com/jotak/netobserv-rivieradev/main/flowcollector-kind.yaml

# Déployer Prometheus
make deploy-prometheus
```

Console Prometheus: http://localhost:9090/
Console Grafana: http://localhost:3000/

Pour éditer la config:

```bash
kubectl edit flowcollector cluster
```

#### Créer le cluster KIND avec podman en root

```bash
sudo kind create cluster
sudo mv /root/.kube/config /home/user/.kube/config-root
sudo chown user:user /home/user/.kube/config-root
export KUBECONFIG=/home/user/.kube/config-root
```

### Autre

Si vous avez OperatorHub, utilisez-le pour installer l'opérateur.

Vous pouvez aussi utiliser [operator-sdk](https://sdk.operatorframework.io/docs/installation/) puis lancer:

```bash
operator-sdk run bundle quay.io/netobserv/network-observability-operator-bundle:v1.6.0 --timeout 5m
```

Autrement, clonez le repo operator et utilisez la Makefile:

```bash
git clone git@github.com:netobserv/network-observability-operator.git
cd network-observability-operator
USER=netobserv make deploy

# Si besoin, pour installer Grafana / Prometheus / Loki:
make deploy-prometheus deploy-grafana deploy-loki

# Configurer NetObserv (resource FlowCollector)
kubectl apply -f https://raw.githubusercontent.com/jotak/netobserv-rivieradev/main/flowcollector-kind.yaml
```

## Déployer des workloads

E.g. demo-mesh-arena:

```bash
kubectl create namespace mesh-arena ; kubectl apply -f https://raw.githubusercontent.com/jotak/demo-mesh-arena/main/quickstart-naked.yml -n mesh-arena
```

DDoS e.g. [hey-ho](https://github.com/jotak/hey-ho):

```bash
./hey-ho.sh -t http://ball.mesh-arena.svc:8080/health -z 5m -r 10 -g hacker -b
```

(Détailler les scénarios)

## Activer des features

Activer les packet drops, latency, DNS, AZ ...

## Usages avancés

- Customiser les métriques, créer des alertes
- Filtrage eBPF, subnet flagging
- CLI, packet capture

## Liens

- Doc OpenShift: https://docs.openshift.com/container-platform/latest/network_observability/installing-operators.html
- GitHub principal: https://github.com/netobserv/network-observability-operator
- OperatorHub: https://operatorhub.io/operator/netobserv-operator
- Compte masto: https://hachyderm.io/@netobserv
