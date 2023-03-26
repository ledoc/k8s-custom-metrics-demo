---
marp: true
theme: default
paginate: true
---
<style>
  section{
    background-image: url("ressources/image/background.png");
    background-size: cover;
  }

  h1 {
    color: #21509c;
    position: absolute;
    top: 15px; 
    left: 28px;
  }
  pre {
    height: max-content;
  }
  p {
    line-height: 38px;
    font-size:26px
  }

  section li {
    line-height: 38px;
    font-size:26px;
    list-style-type: "\21E8\0020";
  }  

</style>  

![bg](ressources/image/titre-pratique-kubernetes.png)


# Ajout de Custom Metrics

## **Architecture de la Démo :**
- Une application en Go qui utilise la library prometheus pour exposer une métrique `http_requests_total`.
- **Prometheus** installé avec un operator.
- **Prometheus Adapter** pour créer la métrique `http_requests_per_second` et proposer la métrique à Kubernetes.
- L'**APIService** `v1beta1.custom.metrics.k8s.io` pour intégrer la métrique de Prometheus Adapter dans Kubernetes.
- HPA basé sur la metric `http_requests_per_second`

---
# Ajout de Custom Metrics

Nécessite que votre appli expose une métrique :

**Exemple en Go :**
```go
const { Counter, register } = require('prom-client');

const counter = new Counter({
    name: 'http_requests_total',
    help: 'Total number of http requests',
    labelNames: ['method'],
});


...

app.get('/metrics', async (req, res) => {
    res.set('Content-Type', register.contentType);
    res.end(await register.getSingleMetricAsString('http_requests_total'));
});
```

---
# Ajout de Custom Metrics

Mise en place de Prometheus et du Prometheus Adapter.

### **Configmap du Prometheus Adapter :** 
```yaml
  config.yaml: |
    rules:
        - seriesQuery: 'http_requests_total{namespace!="",pod!=""}'
        resources:
            overrides:
            namespace:
                resource: namespace
            pod: 
                resource: pod
        name:
            matches: "^(.*)_total"
            as: "${1}_per_second"
        metricsQuery: 'sum(rate(<<.Series>>{<<.LabelMatchers>>}[2m])) by (<<.GroupBy>>)'
```

---
# Ajout de Custom Metrics

### **Explication de la ConfigMap :**
- **seriesQuery** : requête prometheus qui récupére la métrique sur tous les Namespaces et tous les Pods. 
- **resources.overrides** : lien entre les paramètres (namespace!="",pod!="") de la métrique Prometheus et les ressources Kubernetes (api-resource, pod et namespace, v1)

- La métrique `_per_second` est calculée à partir de la métrique `_total` avec la `metricsQuery` nécessaire.

---
# Ajout de Custom Metrics

Définition d'un **APIService** `v1beta1.custom.metrics.k8s.io` qui indique où aller récupérer les custom metrics dans `spec.service` :

```yaml
kind: APIService
metadata:
  name: v1beta1.custom.metrics.k8s.io
  labels:
    app: prometheus-adapter
spec:
  service:
    name: custom-metrics-prometheus-adapter
    namespace: monitoring
  group: custom.metrics.k8s.io
  version: v1beta1
```
---
# Ajout de Custom Metrics

Définition d'un HPA utilisant la custom metric  `http_requests_per_second` :

```yaml
kind: HorizontalPodAutoscaler
apiVersion: autoscaling/v2beta2
metadata:
  name: http
  namespace: demo
spec:
...
  metrics:
  # "Pods" metric, prend la moyenne de la metrique sur tout les pods de la cible
  - type: Pods
    pods:
      metric:
        name: http_requests_per_second
      target:
        # 500 milli-requests /sec = 1 request pour 2 sec
        type: AverageValue
        averageValue: 500m
```