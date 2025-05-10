# Argo CD Configuration
### Add a Kubernetes Cluster to Argo CD
Get the Kubernetes context name:
 ```
 kubectl config get-contexts -o name
 ```
Add the cluster to Argo CD:
 ```
 argocd cluster add <your_cluster_context>
 ```

 Replace `<your_cluster_context>` with the output from the previous command (e.g.,  `arn:aws:eks:region:account-id:cluster/cluster-name`).  Confirm by typing 'Y'.
### Set the Argo CD Namespace Context (Optional)

```
kubectl config set-context --current --namespace=argocd
```

This command sets the current namespace in your kubectl context to argocd. This can be useful if you're frequently working with resources in that namespace, but it's not strictly required for Argo CD operation.

### Create an Argo CD Application

#### With ArgoCD CLI:

# Guestbook Application Deployment with Argo CD

This document outlines how to deploy the Guestbook application using Argo CD.

## Prerequisites

* An operational Kubernetes cluster.
* Argo CD installed and configured within your Kubernetes cluster.
* Access to the command-line interface (`kubectl`) configured to interact with your Kubernetes cluster.
* The Argo CD command-line interface (`argocd`) installed and configured.

## Deployment Steps

### Create the Argo CD Application

To create the Guestbook application within Argo CD, execute the following command:

```
argocd app create guestbook \
  --repo https://github.com/iamsunnyrpa/kubernetes.git \
  --path ArgoCD/Sample_Application \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace default
```
### Explanation of the command:

* argocd app create guestbook: This command instructs Argo CD to create a new application named guestbook.
* --repo https://github.com/iamsunnyrpa/kubernetes.git: This specifies the Git repository containing the application manifests.
* --path ArgoCD/Sample_Application: This indicates the path within the Git repository where the application manifests are located.
* --dest-server https://kubernetes.default.svc: This defines the Kubernetes API server where the application will be deployed. In this case, it targets the in-cluster Kubernetes API server.
* --dest-namespace default: This specifies the Kubernetes namespace where the application resources will be deployed.
### (Optional) Manually Sync the Application
If you have configured the Argo CD application to not automatically synchronize, you can manually trigger a synchronization using the following command:


```
argocd app sync guestbook
```

This command tells Argo CD to reconcile the state of the guestbook application with the desired state defined in the Git repository.

### Verification:
After either the automatic or manual synchronization, Argo CD will begin deploying the Guestbook application to the default namespace of your Kubernetes cluster. You can monitor the deployment status using the Argo CD UI or the following CLI command:


```
argocd app get guestbook
```
This will display the current status and health of the guestbook application resources.

### With ArgoCD UI:

* Access your Load Balancer DNS in a web browser. This can be obtained from the **argocd-server** service within the **argocd** namespace.
* Enter your user credentials to log in.
* Upon successful login, click on the "+ New App" button, as illustrated below:
![add_new.png](ArgoCD/images/add_new.png)
* Provide the Application Name, Project, and select the desired Sync Policy:
![General.png)](ArgoCD/images/General.png)
* Configure the source repository with the following details:
  - Repository URL: https://github.com/iamsunnyrpa/kubernetes.git
  - Path: ArgoCD/Sample_Application
![source.png](ArgoCD/images/source.png)
* On the subsequent page, specify the target cluster and namespace:
![Destination.png](ArgoCD/images/Destination.png)

* Once it created, on the Applications page, click on Sync button of the guestbook application.



# Troubleshooting and Error Resolution
### Connection Failure to Kubernetes API Server
Observed Error:

```
~ $ argocd app get guestbook --server [https://kubernetes.default.svc](https://kubernetes.default.svc)
{"level":"fatal","msg":"Failed to establish connection to [https://kubernetes.default.svc](https://kubernetes.default.svc): dial tcp: lookup tcp///kubernetes.default.svc: unknown port","time":"2025-05-09T14:12:52Z"}
~ $ argocd app get guestbook --server [https://kubernetes.default.svc:443](https://kubernetes.default.svc:443)
{"level":"fatal","msg":"Failed to establish connection to [https://kubernetes.default.svc:443](https://kubernetes.default.svc:443): dial tcp: address [https://kubernetes.default.svc:443](https://kubernetes.default.svc:443): too many colons in address","time":"2025-05-09T14:14:07Z"}
```

#### Root Cause Analysis:

These errors indicate that the argocd CLI is unable to connect to the Kubernetes API server. The first error suggests a DNS resolution issue or an incorrect port specification. The second error points to a malformed URL with too many colons.

#### Resolution:

To establish a connection, you might need to expose the Argo CD server.  A quick way to do this for local testing is port-forwarding:

```
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

This forwards local port 8080 to port 443 of the argocd-server service. You can then access the server via localhost:8080.  For production, use a LoadBalancer or Ingress.

### Argo CD Server Address Unspecified

#### Observed Error:

~ $ argocd app get guestbook
{"level":"fatal","msg":"Argo CD server address unspecified","time":"2025-05-09T14:04:33Z"}
~ $ argocd app sync guestbook
{"level":"fatal","msg":"Argo CD server address unspecified","time":"2025-05-09T14:04:55Z"}
Root Cause Analysis:

The argocd CLI doesn't know the address of the Argo CD API server.  This is typically configured with the argocd login command.

#### Resolution:

Determine Argo CD Server Address: Use kubectl get svc -n argocd to find the EXTERNAL-IP (or NodePort/ClusterIP if applicable).

Login to Argo CD:


```
argocd login <argocd-server-address>
```

Replace <argocd-server-address> with the correct address. You'll be prompted for the username (admin) and password (obtained with argocd admin initial-password -n argocd).

### Important Notes:

**Security:** Port forwarding is for development/testing. Production environments require robust security (e.g., TLS, authentication).
**Production:** Consider high availability, monitoring, and GitOps best practices for production deployments.