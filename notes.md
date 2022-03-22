# Custom metric http_requests_total 

## Source 
https://github.com/antonputra/tutorials/tree/main/lessons/073

## Résumé

Emission d'une metric `http_requests_total`  
Création d'un custom metric (v1beta1.custom.metrics.k8s.io) `http_requests_per_second` avec **prometheus adapter**

Définition d'un HPA réagissant à cette metric


## Commandes pour jouer une démo :  
```bash

kubectl apply -f 1-namespaces

#############################################
# Installation Prometheus Operator avec CRD #
#############################################
kubectl apply -f 2-prometheus-operator-crd
kubectl apply -f 3-prometheus-operator

kubectl get pods -n monitoring
kubectl logs -l app.kubernetes.io/name=prometheus-operator -n monitoring

###########################
# Installation Prometheus #
###########################
kubectl apply -f 4-prometheus

kubectl get pods -n monitoring
kubectl logs -l app.kubernetes.io/instance=prometheus  -n monitoring
kubectl port-forward svc/prometheus-operated 9090 -n monitoring
# http://localhost:9090
# query sur "http" pas encore présent

kubectl apply -f 5-demo

# Dans un autre terminal (garder actif jusqu'à la fin)
kubectl port-forward svc/express 8081 -n demo

# Dans le terminal principal
curl localhost:8081/fibonacci -H "Content-Type: application/json" -d '{"number": 10}'

kubectl get hpa -n demo
# trouve pas encore la metrics
kubectl describe hpa http -n demo

# Pas encore present 
kubectl get --raw /apis/custom.metrics.k8s.io/v1beta1 | jq

########################################################
# Installation de Prometheus Adapter et custom metrics #
########################################################
kubectl apply -f 6-prometheus-adapter/0-adapter
kubectl apply -f 6-prometheus-adapter/1-custom-metrics
kubectl apply -f 6-prometheus-adapter/2-resource-metrics

kubectl get --raw /apis/custom.metrics.k8s.io/v1beta1 | jq

# Check http
kubectl port-forward svc/prometheus-operated 9090 -n monitoring
# http://localhost:9090

# Dans 2 autres terminaux
watch -n 1 -t kubectl get hpa -n demo
watch -n 1 -t kubectl get pods -n demo

# Dans le terminal principal (spammer !!!)
curl localhost:8081/fibonacci -H "Content-Type: application/json" -d '{"number": 10}'

# Suppression
kubectl delete -f 2-prometheus-operator-crd
kubectl delete -f 3-prometheus-operator
kubectl delete -f 4-prometheus
kubectl delete -f 5-demo
kubectl delete -f 6-prometheus-adapter/0-adapter
kubectl delete -f 6-prometheus-adapter/1-custom-metrics
kubectl delete -f 6-prometheus-adapter/2-resource-metrics
kubectl delete ns demo monitoring
```