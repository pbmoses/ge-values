# This repo was created to deploy the Grafana (viz) Enterprise Helm chart from ArgoCD with custom values utilizing multiple sources. 
To note: These repos were developed for rapid redeployment of the Grafana Enterprise stack (there will be one for each of LGTM )based on the repo maintainers knowledge and experience around Kubernetes, this is not official Grafana Documentation. These can also be a basis for the OSS version. 
A rapidly redployable stack should be present in any enterprise environment, these examples can be used as a foundation but should not be the be all end all of MTTR. 

## The Why
Efficiency, repeatability and accountability. Utilizing GitOps approaches, one can (theorehtically) lock down a cluster with all actions that take place doing so only after a peer review and merge request has been completed and approved. I have worked with large enterprise customers where only a handful of people had direct cluster access, all changes took place vis GitOps; Branch, edit, peer review, merge request and move forward. The "oops" moments were not completely gone but were reduced changes (if not done with direct cluster/machine access) were auditable in a single place (Git). 

## End Goals
In the end, I would like to setup 3 demo paths, the stack, the datasources and the dashboards. As will all things OpenSource, there are many ways these can be done. The end objective is to be sure you learn and come away from this with more confidence and knowledge for your day to day computing experiences. 

The demos rely on 2 base ites:
- A working Kubernetes cluster of your choice.
- ArgoCD installed and working properly.
  
## For a minimal Grafana Enterprise functioning setup, you will need 4 manifests:

- A secret for your admin user and password
- A secret for your license file
- The ArgoCD application utilizing mulltiple sources (one pointing to the Grafana Helm chart, one to this directory with overrides)
- Your Helm overrides file

Examples of each of these can be found below. [Secret storage in Kubernetes is not natively secure](https://kubernetes.io/docs/concepts/configuration/secret/#:~:text=%23%20values%20are%20base64,level%20of%20confidentiality), secrets are merely base64 encoded and should never be stored in Git. 
[The ExternalSecretOperator](https://external-secrets.io/latest/) paired with a Vault back end (or other secure secrets manager) is recommended. There are multiple approaches to external secrets, [this](https://www.redhat.com/en/blog/external-secrets-with-hashicorp-vault) linked article is an aging but good foundational knowledge for this approach. For this base tutorial, secrets are created imperatively by the user but this brings me to another point; [Kubernetes is a declarative system](https://kubernetes.io/docs/tasks/manage-kubernetes-objects/declarative-config/), imperative approaches can be used for testing and learning but ultimately declarative approaches are desired. The focus of these repos is not the ESO or Vault or imperative vs declarative approaches but it is important to call these out for general knowledge. 

### The admin user secret
```bash
apiVersion: v1
data:
  adminPassword: <your pass base 64 encoded>
  adminUser: <your admin user base64 encoded>
kind: Secret
metadata:
  name: admin-user
  namespace: grafana-prod
type: Opaque
```

### The license file secret
```bash
apiVersion: v1
data:
  grafana-prod.jwt: <your license file base64 encoded>
kind: Secret
metadata:
  name: ge-license
  namespace: grafana-prod
type: Opaque
```

### The ArgoCD Application
```bash
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: grafana-enterprise
  namespace: argocd
spec:
  destination:  ### where this will be deployed on your cluster
    namespace: grafana-prod    
    server: https://kubernetes.default.svc
  sources:
###maintained main Helm chart directory
    - repoURL: https://grafana.github.io/helm-charts
###chart version
      targetRevision: 8.5.2
      chart: grafana
      helm:
###this will be YOUR git repo path with your overrides file
        valueFiles:
          - $values/charts/grafana/ge-overrides.yaml
### Your Git Repo 
    - repoURL: https://github.com/pbmoses/helm-charts.git
      targetRevision: HEAD 
      ref: values
  project: default
```
### The overrides file
[Values that you have determined are best for your environment](https://github.com/pbmoses/helm-charts/tree/main/charts/grafana/ge-overrides.yaml)

Deploy each of the secrets `kubectl create -f <manifest>`. You can either apply the ArgoCD application imperatively `kubectl create -f application.yaml`or also place this in a repo to allow ArgoCD to sync from it. Your values file should be present in the source that ArgoCD is pointing to. 

Upon successful syncronization and deployment of the Grafana front end, you can retrieve the initial admin password with the following command: `kubectl get secret admin-user -o jsonpath="{.data.adminPassword}" | base64 -d`

### The ArgoCD base application deployed and sync'd
<img width="1253" alt="Screenshot 2024-10-12 at 9 48 37 AM" src="https://github.com/user-attachments/assets/d403af90-8af5-4151-8091-a693f0dd4257">


### The Grafana Enterprise application deployed and sync'd
<img width="1294" alt="Screenshot 2024-10-12 at 9 33 41 AM" src="https://github.com/user-attachments/assets/3b381886-ee6b-4a5d-9b0b-244c90cedb20">


### A deployed Grafana Enterprise front end with successful login 
<img width="1444" alt="Screenshot 2024-10-12 at 9 08 44 AM" src="https://github.com/user-attachments/assets/a7242f70-2649-4e50-9518-01307dcefb1c">

### Following this, check in for similar demos for Mimir, Loki, Tempo, Datasources and Dashboards

* Disclaimer. Best efforts are approached and we're all human. Use as a foundation, deploy, verify and approve in your own environment with developed best practices  similar to a pattern of sandbox/dev, staging and prod (or similar). 
