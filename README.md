# The purpose of this repo is to deploy the Grafana (viz) Enterprise Helm chart from ArgoCD with custom values utilizing multiple sources. 
To note: These repose were developed for rapid redeployment of the Grafana Enterprise stack based on the repo maintainers knowledge and experience around Kubernetes, this is not official Grafana Documentation. 
A rapidly redployable stack should be present in enterprise environments, these methods can be used as a foundation but should not be the be all end all of MTTR. 

The demos rely on 2 base ites:
- A working Kubernetes cluster of your choice.
- ArgoCD installed and working properly.
  
## For a minimal Grafana Enterprise functioning setup, you will need 4 manifests:

- A secret for your admin user and password
- A secret for your license file
- The ArgoCD application utilizing mulltiple sources (one pointing to the Grafana Helm chart, one to this directory with overrides)
- Your Helm overrides file

Examples of each of these can be found below. [Secret storage in Kubernetes is not natively secure](https://kubernetes.io/docs/concepts/configuration/secret/#:~:text=%23%20values%20are%20base64,level%20of%20confidentiality), secrets are merely base64 encoded and should never be stored in Git. 
[The ExternalSecretOperator](https://external-secrets.io/latest/) paired with a Vault back bend (or other secure secrets manager) is recommended. For this base tutorial, secrets are created imperatively by the user but this brings me to another point; [Kubernetes is a declarative system](https://kubernetes.io/docs/tasks/manage-kubernetes-objects/declarative-config/), imperative approaches can be used for testing and learning but ultimately declarative approaches are desired. 

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


