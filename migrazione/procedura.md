# Procedura per la Raccolta di Informazioni sui Nodi Kubernetes

Questa procedura descrive come deployare un DaemonSet su un cluster Kubernetes per raccogliere informazioni specifiche da ogni nodo.

## Prerequisiti

- Accesso a un cluster Kubernetes.
- `kubectl` installato e configurato per comunicare con il cluster (es. tramite la variabile d'ambiente `KUBECONFIG`).

## Passaggi

### 1. Deploy del DaemonSet

Questo passaggio deploya il DaemonSet `node-config-collector` nel namespace `kube-system`. Il DaemonSet avvierà un pod su ogni nodo del cluster.

Esegui il seguente comando dalla directory `migrazione`:
```bash
kubectl apply -f node-config-collector.yaml
```

### 2. Verifica del Deploy

Assicurati che i pod del DaemonSet siano in esecuzione correttamente.

```bash
kubectl get pods -n kube-system -l name=node-config-collector
```
Dovresti vedere un pod per ogni nodo del cluster con lo stato `Running`.

### 3. Raccolta delle Informazioni

Una volta che i pod sono in esecuzione, puoi visualizzare i log di ciascuno per vedere le informazioni raccolte.

Il seguente script itera su tutti i pod del DaemonSet e stampa i loro log:

```bash
#!/bin/bash

# Ottieni la lista di tutti i pod del collector
POD_LIST=$(kubectl get pods -n kube-system -l name=node-config-collector -o jsonpath='{.items[*].metadata.name}')

if [ -z "$POD_LIST" ]; then
  echo "Nessun pod del node-config-collector trovato."
  exit 1
fi

# Itera su ogni pod e stampa i suoi log
for pod in $POD_LIST; do
  echo "================================================================="
  echo "LOGS PER IL POD: $pod"
  echo "================================================================="
  kubectl logs -n kube-system $pod > /tmp/$pod.log
  cat /tmp/$pod.log
  echo
done
```

### 4. Pulizia delle Risorse

Dopo aver raccolto tutte le informazioni necessarie, è importante rimuovere il DaemonSet per non lasciare risorse non necessarie nel cluster.

Esegui il seguente comando dalla directory `migrazione`:
```bash
kubectl delete -f node-config-collector.yaml
```
