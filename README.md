# kubernetes
A comprehensive collection of resources, guides, and best practices for container orchestration with Kubernetes.
|A|B|
|--|--|
|*aws eks update-kubeconfig —region region-code —name my-cluster*|To set the config for the mentioned cluster name in region.|
*kubectl config current-context*|To know the current EKS cluster configurations.|
*kubectl config get-contexts*|To know the details about all EKS cluster configurations.|
*kubectl config view*|To know the detailed view of EKS cluster config.|
*kubectl config use-context context-name*|Switch to a different EKS cluster config.|
*kubectl config view —minify —output 'jsonpath={..user}{"\n"}'*|To see which user/identity you're authenticated.|
