# argocd

1. Setup K8S cluster.
2. Add argocd repo: 
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh

helm repo add argo https://argoproj.github.io/argo-helm
helm repo update
helm search repo argocd
helm show values argo/argo-cd --version <version> > argocd-defaults.yml
helm install argocd -n argocd --create-namespace argo/argo-cd --version 3.35.4 -f terraform/values/argocd.yaml
helm list -A

3. check argocd pods
kubectl get pods -n argocd
kubectl get secrets -n argocd
kubectl get secrets argocd-initial-admin-secret -o yaml -n argocd 
echo <secret> | base64 -d
kubectl port-forward svc/argocd-server -n argocd 8080:80

======== OR ============

You need to create a namespace for argocd.

kubectl create ns argocd

and then choose one of the below options :

1. Non-HA:
a. cluster-admin privileges: 
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
b. namespace level privileges: 
kubectl apply -n argocd -f https://github.com/argoproj/argo-cd/raw/stable/manifests/namespace-install.yaml

2. HA:
a. cluster-admin privileges: 
kubectl apply -n argocd -f https://github.com/argoproj/argo-cd/raw/stable/manifests/ha/install.yaml
b. namespace level privileges: 
kubectl apply -n argocd -f https://github.com/argoproj/argo-cd/raw/stable/manifests/ha/namespace-install.yaml

3. Light installation "Core"
kubectl apply -n argocd -f https://github.com/argoproj/argo-cd/raw/stable/manifests/core-install.yaml

4. Helm chart: 
kubectl apply -n argocd -f https://github.com/argoproj/argo-helm/tree/main/charts/argo-cd

Change the argocd server service type to LoadBalancer or NodePort-
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'

---> After Installation process <---
Install the argocd CLI 
brew install argocd

Get the initial admin password, login and change the password:
kubectl get secret -n argocd
kubectl get secret -n argocd argocd-initial-admin-secret -o yaml
OR
argocd admin initial-password -n argocd
argocd login <ARGOCD_SERVER>:port
argocd account update-password

Access the UI:
By default ArgoCD server is not exposed with external endpoint.
It can be exposed by using -
a. Service: LoadBalancer
b. Ingress
c. Port Forward to access locally -
kubectl port-forward svc/argocd-server -n argocd 8080:443
open browser https://localhost:8080/
