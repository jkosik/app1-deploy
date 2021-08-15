# ArgoCD Application Deployment
This project contains ArgoCD objects deployed during onboarding of the new Application.

Every application should follow DEV-STAGE-PROD workflow and DevSecOps should ensure all environments are ready when new application project starts.
To save resources, it is acceptable to condense DEV and STAGE to one non-prod Kubernetes cluster and separate using namespaces.
Potential risk of merging DEV and STAGE into one non-prod cluster are: 1) Chance of overwriting cluster-wide objects and CRDs between namespaces and 2) Potentially confusing parametrization.

## ArgoCD self-configuration
Latest versions of ArgoCD allow declarative GitOps CD for ArgoCD itself and we are not limited only to submit Application resources.
We can manage multiple ArgoCD objects via GitOps, including ArgoCD configuration ConfigMaps, Projects or Repositores. Check [Operator manual](https://argo-cd.readthedocs.io/en/latest/operator-manual/declarative-setup/) for details and caveats.

