# Argo CD Installation and Configuration Guide

This document provides a detailed guide to installing, configuring, and troubleshooting Argo CD based on the provided script.

## Installation

###  Install Argo CD

   ```
   kubectl create namespace argocd
   kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
   ```
   (https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml)

This set of commands first creates a dedicated namespace argocd for the Argo CD deployment and then applies the official installation manifests to that namespace. This will deploy all the necessary Argo CD components in your Kubernetes cluster.

2. Verify Installation

```
kubectl get all -n argocd
```

Use this command to check the status of all the pods, services, and other resources created in the argocd namespace. This will confirm that the installation was successful and that all the components are running.

Argo CD CLI Installation
### Install Argo CD CLI

```
curl -sSL -o argocd-linux-amd64 [https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64](https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64)
sudo install -m 555 argocd-linux-amd64 /usr/local/bin/argocd
rm argocd-linux-amd64
```

These commands download the Argo CD CLI for Linux, make it executable, and move it to /usr/local/bin so that it's available system-wide.  The downloaded file is then removed.

### Argo CD Server Configuration
Expose the Argo CD Server

```
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
```

This command patches the argocd-server service to change its type to LoadBalancer. This is crucial for accessing the Argo CD UI from outside the Kubernetes cluster in cloud environments.  If you are running in a local environment like Minikube, you might use NodePort or port-forward instead.

5. Get Argo CD Server Details

```
kubectl get svc -n argocd
```

This command retrieves the details of all services running in the argocd namespace.  The output will include the EXTERNAL-IP (if a LoadBalancer is used) or NODEPORT (if NodePort is used) of the argocd-server service, which is needed to access the Argo CD UI.

Example Output:

NAME                                      TYPE           CLUSTER-IP       EXTERNAL-IP                                                              PORT(S)                      AGE
argocd-applicationset-controller          ClusterIP      172.20.132.182   <none>                                                                   7000/TCP,8080/TCP            4m15s
argocd-dex-server                         ClusterIP      172.20.4.150     <none>                                                                   5556/TCP,5557/TCP,5558/TCP   4m15s
argocd-metrics                            ClusterIP      172.20.5.242     <none>                                                                   8082/TCP                     4m15s
argocd-notifications-controller-metrics   ClusterIP      172.20.96.215    <none>                                                                   9001/TCP                     4m15s
argocd-redis                              ClusterIP      172.20.118.120   <none>                                                                   6379/TCP                     4m15s
argocd-repo-server                        ClusterIP      172.20.66.247    <none>                                                                   8081/TCP,8084/TCP            4m15s
argocd-server                             LoadBalancer   172.20.172.144   <LB DNS>                                                                 80:32693/TCP,443:31208/TCP   4m15s
argocd-server-metrics                     ClusterIP      172.20.250.94    <none>                                                                   8083/TCP                     4m15s

## Retrieve the Default Admin Password
The default username is admin.
To get the initial password, run:
 ```
 argocd admin initial-password -n argocd
 ```

### Access the Argo CD UI
Visit the DNS or IP address of the argocd-server service in your web browser.

