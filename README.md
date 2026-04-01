# 🔐 Security Lab — Quick Reference

## Part 1 — RBAC
# Role & RoleBinding (alice, read-only pods)
kubectl create namespace dev
kubectl create role pod-reader --verb=get,list,watch --resource=pods -n dev
kubectl create rolebinding read-pods --role=pod-reader --user=alice -n dev
kubectl auth can-i get pods -n dev --as=alice     # yes
kubectl auth can-i delete pods -n dev --as=alice  # no

# ClusterRole & ClusterRoleBinding (bob, cluster nodes)
kubectl create clusterrole node-viewer --verb=get,list --resource=nodes
kubectl create clusterrolebinding view-nodes --clusterrole=node-viewer --user=bob
kubectl auth can-i list nodes --as=bob  # yes
kubectl auth can-i list pods --as=bob   # no
# Difference: Role = namespace-scoped, ClusterRole = cluster-wide

# ServiceAccount (app-sa)
kubectl create serviceaccount app-sa -n dev
kubectl create rolebinding sa-read-pods --role=pod-reader --serviceaccount=dev:app-sa -n dev
kubectl apply -f sa-demo.yaml
kubectl exec sa-demo -n dev -- ls /var/run/secrets/kubernetes.io/serviceaccount/
# files: ca.crt  namespace  token

## Part 2 — SecurityContext & NetworkPolicy
# SecurityContext (non-root, read-only filesystem)
securityContext:
  runAsNonRoot: true
  runAsUser: 1000
containers:
  - name: app
    image: busybox
    command: ['sleep','3600']
    securityContext:
      readOnlyRootFilesystem: true
      capabilities:
        drop: [ALL]

# NetworkPolicy
kubectl run frontend --image=nginx -n dev --labels=app=frontend
kubectl run backend  --image=nginx -n dev --labels=app=backend
kubectl apply -f deny-all.yaml       # block all ingress
kubectl apply -f allow-frontend.yaml # allow frontend -> backend on port 80

## Part 3 — kubeconfig
kubectl config current-context
kubectl config get-contexts
kubectl config view
cat ~/.kube/config | grep -E 'cluster|user|namespace'
# Sections: clusters, users, contexts
# Context ties: cluster + user + namespace
