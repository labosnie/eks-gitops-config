# Project Tracker

## Projet

Plateforme GitOps Kubernetes sur AWS EKS.

## Statut global

Phase d'apprentissage Kubernetes local.

## Environnement validé

- Windows 11
- WSL2
- Ubuntu 24.04
- Docker Desktop opérationnel depuis Ubuntu
- kubectl v1.35.6
- kind v0.32.0
- cluster Kubernetes v1.35.5

## Architecture locale actuelle

- 1 control plane
- 1 worker
- cluster : `gitops-local`
- namespace applicatif : `incident-platform`

## Étapes terminées

- [x] Préparation de WSL2
- [x] Validation de Docker
- [x] Installation de kubectl
- [x] Installation de kind
- [x] Création du cluster local
- [x] Validation des nœuds et Pods système
- [x] Création déclarative du namespace `incident-platform`

## Décisions d'architecture

### ADR-001 — Utilisation de kind

kind est utilisé pour l'apprentissage et la validation locale avant EKS.

### ADR-002 — Cluster local léger

Le cluster commence avec un control plane et un worker afin de limiter la consommation de ressources.

### ADR-003 — Séparation en trois dépôts

- application ;
- configuration GitOps ;
- infrastructure Terraform.

### ADR-004 — Kubernetes 1.35

La branche Kubernetes 1.35 est utilisée localement afin de rester proche de la future cible EKS.

## Prochaine étape

Créer et observer un premier Pod, puis comprendre pourquoi un Pod seul n'est pas suffisant pour une application professionnelle.
