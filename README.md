# NetObserv au RivieraDev 2024!

Ce repo contient les instructions et resources utiles au déroulement du [Deep Dive NetObserv](https://www.rivieradev.fr/session/226).

## Installation

### OpenShift

### KIND

- cloner: `git clone git@github.com:netobserv/network-observability-operator.git`
- `kind create cluster` (avec sudo si nécessaire (podman...) => dans ce cas, récupérer le /root/.kube/config créé)
- `USER=netobserv make deploy-kind`
- `make deploy-grafana deploy-loki`
- `kubectl apply -f https://raw.githubusercontent.com/jotak/netobserv-rivieradev/main/flowcollector.yaml`

### Autre


## Liens

- Doc OpenShift: https://docs.openshift.com/container-platform/latest/network_observability/installing-operators.html
- GitHub principal: https://github.com/netobserv/network-observability-operator
- OperatorHub: https://operatorhub.io/operator/netobserv-operator
- Compte masto: https://hachyderm.io/@netobserv
