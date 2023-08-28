# quickstart

Create k3d cluster:
```
# gp-mgmt
k3d cluster create --config k3d/gp-mgmt.yaml

# gp-west
k3d cluster create --config k3d/gp-west.yaml

# gp-east
k3d cluster create --config k3d/gp-west.yaml

```

Note: you will need to change your `/etc/hosts` file to point at the following kubeAPI hosts:
```
127.0.0.1 gp-west.glooplatform.com
127.0.0.1 gp-east.glooplatform.com
127.0.0.1 mgmt.glooplatform.com
```

Install ArgoCD:
```
cd bootstrap-argocd
./install-argocd.sh insecure-rootpath <cluster_context>
```

Check that ArgoCD is up and running:
```
% k get pods -n argocd
NAME                                               READY   STATUS    RESTARTS   AGE
argocd-redis-74f98b85f-54l4w                       1/1     Running   0          5m13s
argocd-applicationset-controller-cff4b447c-t9z49   1/1     Running   0          5m14s
argocd-notifications-controller-64d7d5bcf7-mk8hz   1/1     Running   0          5m14s
argocd-server-7ff7c779f7-sjzj6                     2/2     Running   0          5m13s
argocd-dex-server-6fc57666f4-p4gst                 1/1     Running   0          5m14s
argocd-repo-server-785d677d59-s2hjv                1/1     Running   0          5m13s
argocd-application-controller-0                    1/1     Running   0          5m13s
```

Deploy applications app-of-app, which deploys httpbin
```
kubectl apply -f applications-aoa.yaml
```

Delete cluster:
```
k3d cluster list
k3d cluster delete <cluster name>
```