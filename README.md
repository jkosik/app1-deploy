# ArgoCD Image Updater

## ArgoCD preparation
- Install [ApplicationSet Controller](https://argocd-applicationset.readthedocs.io/en/stable/Getting-Started/). Note: ArgoCD Image Updater do not support ApplicationSets (11/2021)
- Project to build new image tags to test AIU: https://github.com/jkosik/argocd-image-updater-app
- Install and configure [ArgoCD Image Updater](https://argocd-image-updater.readthedocs.io/en/stable/install/start/). At the moment supports only Application resources, not ApplicationSets.

## Manual triggers
`argocd-image-updater` can be triggered also manually:
```
microk8s config > ~/.kube/config
export ARGOCD_TOKEN=<yourtoken>
argocd-image-updater run --applications-api argocd --argocd-server-addr 127.0.0.1:1234 --once --argocd-insecure --loglevel=debug
```

## Annotation example
Example for `app1`. For demonstrating complexity `app1` acts as the Umbrella Helm Chart with Subchart (nginx) and also external dependency (haproxy).

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
