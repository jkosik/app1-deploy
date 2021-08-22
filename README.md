# ArgoCD Application Deployment
This project follows Application of Applications ArgoCD pattern for deploying sample application `app1`.
ArgoCD Application `argocd-app1` deploys all ArgoCD resources from this repository including child Applications referring to separate Git or Helm repositories.

## ArgoCD preparation
- Install [ApplicationSet Controller](https://argocd-applicationset.readthedocs.io/en/stable/Getting-Started/)
- Install and configure [ArgoCD Image Updater](https://argocd-image-updater.readthedocs.io/en/stable/install/start/)

## Application onboarding
1. Create ArgoCD Project `app1`
Latest versions of ArgoCD allow declarative GitOps CD for ArgoCD itself and we are not limited only to submit Application resources.
We can manage multiple ArgoCD objects via GitOps, including ArgoCD configuration ConfigMaps, Projects or Repositores. Check [Operator manual](https://argo-cd.readthedocs.io/en/latest/operator-manual/declarative-setup/) for details and caveats.
ApplicationSet can simplify declaration of `app1` deployment to DEV, STAGE and PROD target clusters.

2. Create ArgoCD Application following this repository's `main` branch.
This repository may contain DEV-STAGE-PROD branches for own code development and testing. However, `app1` Application Owner should interact only with production ArgoCD cluster and consume it as a service.

```
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: argocd-app1
spec:
  project: app1
  destination:
    name: ''
    namespace: argocd
    server: 'https://kubernetes.default.svc'
  source:
    path: .
    repoURL: 'https://github.com/jkosik/argocd-app1.git'
    targetRevision: main
  syncPolicy:
    syncOptions:
      - CreateNamespace=true
```

3. Sync in ArgoCD - set tu manual. Dependency service is configured to be deployed only to DEV environment.

4. Configure argocd-image-updater as descibed in sections above.
To run `argocd-image-updater` locally run (command for port-forwarded ArgoCD on localhost in microk8s):
```
microk8s config > ~/.kube/config
export ARGOCD_TOKEN=<yourtoken>
argocd-image-updater run --applications-api argocd --argocd-server-addr 127.0.0.1:1234 --once --argocd-insecure --loglevel=debug
```

Not using git-creds. dependency Application can not write to docker.io/bitnami
```
  annotations:
    argocd-image-updater.argoproj.io/image-list: nginx=bitnami/nginx:~1.21
    argocd-image-updater.argoproj.io/nginx.force-update: "true"
    argocd-image-updater.argoproj.io/nginx.helm.image-name: image.repository
    argocd-image-updater.argoproj.io/nginx.helm.image-tag: image.tag
    argocd-image-updater.argoproj.io/nginx.update-strategy: name
    argocd-image-updater.argoproj.io/nginx.allow-tags: regexp:^1.21.[0-9]+$
```

```
  annotations:
    argocd-image-updater.argoproj.io/image-list: app1=jkosik/app1,nginx=nginx:~1.21
    argocd-image-updater.argoproj.io/nginx.force-update: "true"
    argocd-image-updater.argoproj.io/nginx.helm.image-name: image.repository
    argocd-image-updater.argoproj.io/nginx.helm.image-tag: image.tag
    argocd-image-updater.argoproj.io/nginx.update-strategy: name
    argocd-image-updater.argoproj.io/nginx.allow-tags: regexp:^1.21.[0-9]+$
    argocd-image-updater.argoproj.io/app1.allow-tags: regexp:^dev-[0-9]+$
    argocd-image-updater.argoproj.io/write-back-method: git:secret:argocd-image-updater/git-creds
```



