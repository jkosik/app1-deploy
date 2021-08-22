# ArgoCD Application Deployment
This project follows Application of Applications ArgoCD pattern for deploying sample application `app1`.
ArgoCD Application `argocd-app1` deploys all ArgoCD resources from this repository including child Applications referring to separate Git or Helm repositories.

## ArgoCD preparation
- Install [ApplicationSet Controller](https://argocd-applicationset.readthedocs.io/en/stable/Getting-Started/).
- Install and configure [ArgoCD Image Updater](https://argocd-image-updater.readthedocs.io/en/stable/install/start/). At the moment supports only Application resources, not ApplicationSets.

## Application onboarding
1. Create ArgoCD Project `app1`
ApplicationSet can simplify declaration of `app1` deployment to DEV, STAGE and PROD target clusters.

2. Create ArgoCD Application following this repository's `main` branch.
This repository may contain DEV-STAGE-PROD branches for own App of Apps code development. However, `app1` Application Owner should interact only with production ArgoCD cluster and consume it as a service.
Deploy App of Apps:

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

3. Sync in ArgoCD.

4. `argocd-image-updater` automatically detects out-of date images based on Annotations defined in Application manifests. Subsequent app Sync triggers rolling restarts.

`argocd-image-updater` can be triggered also manually:
```
microk8s config > ~/.kube/config
export ARGOCD_TOKEN=<yourtoken>
argocd-image-updater run --applications-api argocd --argocd-server-addr 127.0.0.1:1234 --once --argocd-insecure --loglevel=debug
```

#### Annotation example
Example for `app1` containing one parent Helm Chart and on child subchart.

Nginx is part of subchart and image and tag reference in the Annotation must follow path from values.yaml file, i.e. prefixed by nginx (nginx.image.name, nginx.image.tag)
app1.helm.image-name and app1.helm.image-tag contain default path and are used only for clarity (could be removed)
```
  annotations:
    argocd-image-updater.argoproj.io/image-list: app1=jkosik/app1,nginx=nginx:~1.21
    argocd-image-updater.argoproj.io/nginx.force-update: "true"
    argocd-image-updater.argoproj.io/nginx.update-strategy: name # sorts images alphanumerically and picks up latest one.
    argocd-image-updater.argoproj.io/nginx.helm.image-name: nginx.image.name # all references other than image.name and image.tag has to be explicitly defined.
    argocd-image-updater.argoproj.io/nginx.helm.image-tag: nginx.image.tag # all references other than image.name and image.tag has to be explicitly defined.
    argocd-image-updater.argoproj.io/nginx.allow-tags: regexp:^1.21.[0-9]+$
    argocd-image-updater.argoproj.io/app1.force-update: "true"
    argocd-image-updater.argoproj.io/app1.update-strategy: name
    argocd-image-updater.argoproj.io/app1.helm.image-name: image.name # parent Chart uses default references image.name and image.tag. This line can be removed. Used for more clarity.
    argocd-image-updater.argoproj.io/app1.helm.image-tag: image.tag # parent Chart uses default references image.name and image.tag. This line can be removed. Used for more clarity.
    argocd-image-updater.argoproj.io/app1.allow-tags: regexp:^dev-[0-9]+$ # follows ":dev-xxx" pattern
    argocd-image-updater.argoproj.io/app1.ignore-tags: "dev" # ignores default ":dev" tag and compares only allowed-tags.
    #argocd-image-updater.argoproj.io/write-back-method: git:secret:argocd-image-updater/git-creds # used for write-back to git repo storing Helm Charts.
```

To Stop updating image temporarily:
```
argocd-image-updater.argoproj.io/<image_name>.ignore-tags: "*"
```
