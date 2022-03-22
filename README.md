# Custom metric avec Prometheus adapter 

## Source 
https://github.com/antonputra/tutorials/tree/main/lessons/073

## Résumé

- Emission d'une metric `http_requests_total` par l'appli express
- Création d'un custom metric (v1beta1.custom.metrics.k8s.io) `http_requests_per_second` avec **prometheus adapter**

- Définition d'un HPA réagissant à cette metric

**Les commandes à déroulées sont dans le fichier notes.md**