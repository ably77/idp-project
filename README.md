# quickstart

Create Clusters in GKE:
- This step is a prerequisite
- At least two clusters is ideal, named `mgmt` and `cluster1`

Set cluster variables:
```
mgmt="mgmt"
cluster1="cluster1"
```

Install ArgoCD on `mgmt`
```
cd bootstrap-argocd
./install-argocd.sh insecure-rootpath ${mgmt}
```

Check that ArgoCD is up and running:
```
% k get pods -n argocd --context ${mgmt}
NAME                                               READY   STATUS    RESTARTS   AGE
argocd-redis-74f98b85f-54l4w                       1/1     Running   0          5m13s
argocd-applicationset-controller-cff4b447c-t9z49   1/1     Running   0          5m14s
argocd-notifications-controller-64d7d5bcf7-mk8hz   1/1     Running   0          5m14s
argocd-server-7ff7c779f7-sjzj6                     2/2     Running   0          5m13s
argocd-dex-server-6fc57666f4-p4gst                 1/1     Running   0          5m14s
argocd-repo-server-785d677d59-s2hjv                1/1     Running   0          5m13s
argocd-application-controller-0                    1/1     Running   0          5m13s
```

Register remote argocd cluster:
```
argocd cluster add ${cluster1}
```

The output should look similar to below:
```
% argocd cluster add cluster1
WARNING: This will create a service account `argocd-manager` on the cluster referenced by context `cluster1` with full cluster level privileges. Do you want to continue [y/N]? y
INFO[0002] ServiceAccount "argocd-manager" created in namespace "kube-system" 
INFO[0003] ClusterRole "argocd-manager-role" created    
INFO[0003] ClusterRoleBinding "argocd-manager-role-binding" created 
INFO[0008] Created bearer token secret for ServiceAccount "argocd-manager" 
Cluster 'https://34.73.72.254' added
```

Set the remote cluster destination server as a variable:
```
cluster_destination_server="https://34.73.72.254"
```

Deploy argo `ApplicationSet`
```
kubectl apply --context ${mgmt} -f- <<EOF
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: applications
  namespace: argocd
spec:
  generators:
  - list:
      elements:
      - cluster: cluster1
        url: ${cluster_destination_server}
  template:
    metadata:
      name: '{{cluster}}-httpbin'
    spec:
      project: default
      source:
        repoURL: https://github.com/ably77/idp-project
        targetRevision: HEAD
        path: applications/
      destination:
        server: '{{url}}'
      syncPolicy:
        automated:
          prune: true
          selfHeal: true
        syncOptions:
          - ApplyOutOfSyncOnly=true
EOF
```

Verify on `cluster1` that the `httpbin` application has been deployed
```
kubectl get pods -n httpbin --context ${cluster1}
```

Output should look like below:
```
% kubectl get applicationset -n argocd --context ${mgmt}
NAME           AGE
applications   34s

% kubectl get pods -n httpbin --context ${cluster1}
NAME                      READY   STATUS    RESTARTS   AGE
httpbin-9dbd644c7-f5qjd   1/1     Running   0          6s
```