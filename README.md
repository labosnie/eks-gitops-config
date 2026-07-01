# EKS GitOps Configuration

Ce dépôt contient la configuration déclarative de la plateforme GitOps Kubernetes.

## Objectif

Déployer et maintenir une application de gestion d'incidents sur :

- un cluster Kubernetes local avec kind ;
- puis un cluster Amazon EKS.

## Contenu

- `learning/` : exercices Kubernetes progressifs ;
- `platform/` : configuration des composants de la plateforme ;
- `docs/` : suivi, décisions d'architecture et procédures ;
- `charts/` : futurs charts Helm ;
- `argocd/` : future configuration Argo CD ;
- `environments/` : futures configurations par environnement.

## Règles de sécurité

- Aucun secret réel ne doit être enregistré dans Git.
- Aucun mot de passe, token ou accès AWS ne doit être placé dans les manifests.
- Les modifications seront validées avant d'être commitées.
- Git deviendra progressivement la source de vérité du cluster.

## Environnement local

- Ubuntu 24.04 sous WSL2
- Docker Desktop
- kind
- Kubernetes 1.35
- kubectl 1.35
