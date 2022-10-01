# ArgoCD poc

Deploy ArgoCD as a Github App.

Requirements:

- [Kind](https://kind.sigs.k8s.io/docs/user/quick-start/)
- [Helm](https://helm.sh/)
- [ArgoCD](https://argo-cd.readthedocs.io/en/stable/) registered as a Github App with its own key installed in the repo (fill out `configs.credentialTemplates.github-app-creds:` in [argocd/argocd/values.yaml](./argocd/argocd/values.yaml))
  - Github Settings ->  Developer settings -> GitHub Apps: Create new App and get private key.
  - Github Settings ->  Applications: Configure read access to the repo.

## Create Kind cluster

```bash
kind create cluster --config=./kind/cluster.yaml
```

## Install ArgoCD

```bash
helm repo add argo https://argoproj.github.io/argo-helm && helm repo update

kubectl create ns argocd && helm upgrade --install -n argocd argocd argo/argo-cd --version 5.5.7 --values argocd/values.yaml 
```

Access ArgoCD through `localhost:8080` (admin/argocd):

```bash
kubectl port-forward -n argocd svc/argocd-server 8080:80
```

### Change ArgoCD admin pass

Generate password:

```bash
 sudo apt-get install apache2-utils
 ARGO_PWD=argocd
 htpasswd -nbBC 10 "" $ARGO_PWD | tr -d ':\n' | sed 's/$2y/$2a/'
```

Change the value of `configs.secret.argocdServerAdminPassword:` in [argocd/argocd/values.yaml](./argocd/argocd/values.yaml) with this password.

## Deploy ArgoCD Applications

```bash
# Deploy manifests from master branch in this repo in argocd/argocd/argo-apps/nginx-vanilla/ path.
# Update argocd/argo-apps/nginx-vanilla.yaml spec.source to read from other repo/branch/path (will require installing ArgoCD as github app if you change the repo)
kubectl apply -f argocd/argo-apps/nginx-vanilla.yaml

# Deploy Bitnami's Nginx Helm chart
kubectl apply -f argocd/argo-apps/nginx-bitnami-helm.yaml
```

## Delete Kind cluster

```bash
kind delete cluster --name argocd-cluster
```

## Extra Resources

ArgoCon22 Workshops:

- [Akuity Best Practices](https://github.com/argocon2022-workshop).
- [ArgoCD/Rollouts 101/201](https://github.com/argocon22Workshop/ArgoCDRollouts)
