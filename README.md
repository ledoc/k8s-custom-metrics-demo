# Custom metric avec Prometheus adapter 

## Source 
https://github.com/antonputra/tutorials/tree/main/lessons/073

## Résumé

- Emission d'une metric `http_requests_total` par l'appli express
- Enregistrement de la metric par un prometheus (install bof)
- Création d'un custom metric (v1beta1.custom.metrics.k8s.io) `http_requests_per_second` avec un **prometheus adapter**

- Définition d'un HPA réagissant à cette metric

**Les commandes à dérouler sont dans le fichier notes.md**