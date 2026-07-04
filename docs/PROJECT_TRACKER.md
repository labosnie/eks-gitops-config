# Incident 001 — Contrôleur Ingress sur le mauvais nœud kind

## Résumé

Lors de la configuration de l’Ingress local, l’application ne répondait pas via `localhost:8080`.

La commande suivante échouait :

```bash
curl -H "Host: hello.localhost" http://localhost:8080
```

Erreur obtenue :

```text
curl: (56) Recv failure: Connection reset by peer
```

Le problème venait du fait que le trafic entrait dans le cluster par le nœud `control-plane`, alors que le contrôleur Ingress NGINX tournait sur le nœud `worker`.

---

## Contexte

Cluster kind local :

```text
gitops-local-control-plane
gitops-local-worker
```

Ports exposés par kind :

```text
localhost:8080 → gitops-local-control-plane:80
localhost:8443 → gitops-local-control-plane:443
```

Application :

```text
Namespace  : incident-platform
Deployment : hello-deployment
Service    : hello-service
Ingress    : hello-ingress
```

Contrôleur Ingress :

```text
Namespace : ingress-nginx
Pod       : ingress-nginx-controller
```

---

## Diagnostic

L’application était saine :

```bash
kubectl get deployment,service,pods -n incident-platform
```

Le Deployment était disponible, le Service existait et les Pods étaient `Running`.

Le Service trouvait bien les Pods :

```bash
kubectl get endpointslices -n incident-platform \
  -l kubernetes.io/service-name=hello-service -o wide
```

L’Ingress pointait correctement vers le Service :

```bash
kubectl describe ingress hello-ingress -n incident-platform
```

Résultat important :

```text
hello.localhost / → hello-service:80
```

Le problème ne venait donc pas du Deployment, du Service, des labels ou de l’Ingress.

La cause a été trouvée avec :

```bash
kubectl get pods -n ingress-nginx -o wide
```

État problématique :

```text
ingress-nginx-controller → gitops-local-worker
```

---

## Cause racine

Dans kind, les ports exposés avec `extraPortMappings` sont attachés à un nœud précis.

Dans notre cas :

```text
localhost:8080 → gitops-local-control-plane:80
```

Mais le contrôleur Ingress tournait sur :

```text
gitops-local-worker
```

Le trafic arrivait donc sur le nœud `control-plane`, alors que NGINX tournait sur le nœud `worker`.

---

## Correction

Ajouter un label au nœud `control-plane` :

```bash
kubectl label node gitops-local-control-plane ingress-ready=true --overwrite
```

Forcer le contrôleur Ingress à tourner sur ce nœud :

```bash
kubectl patch deployment ingress-nginx-controller \
  -n ingress-nginx \
  --type='merge' \
  -p '{"spec":{"template":{"spec":{"nodeSelector":{"ingress-ready":"true"},"tolerations":[{"key":"node-role.kubernetes.io/control-plane","operator":"Exists","effect":"NoSchedule"},{"key":"node-role.kubernetes.io/master","operator":"Exists","effect":"NoSchedule"}]}}}}'
```

Vérifier le redéploiement :

```bash
kubectl rollout status deployment/ingress-nginx-controller \
  -n ingress-nginx --timeout=120s
```

---

## Validation

Le contrôleur Ingress tourne maintenant sur le bon nœud :

```bash
kubectl get pods -n ingress-nginx -o wide
```

Résultat attendu :

```text
ingress-nginx-controller → gitops-local-control-plane
```

Test réussi :

```bash
curl -H "Host: hello.localhost" http://localhost:8080
```

Résultat :

```text
Bonjour depuis le Pod : hello-deployment-xxxxx
```

---

## Leçons apprises

* Dans kind, un port exposé est lié à un nœud précis.
* Un Pod `Running` ne garantit pas que tout le flux réseau fonctionne.
* Il faut vérifier toute la chaîne : Ingress Controller → Ingress → Service → EndpointSlice → Pods.
* Le contrôleur Ingress doit être placé sur le nœud qui reçoit le trafic local.
* La correction actuelle est manuelle ; elle devra plus tard être rendue reproductible avec GitOps, Helm ou Kustomize.
