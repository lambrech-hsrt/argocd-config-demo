# argocd-config-demo
Argo CD Config Demo Repository with user management and RBAC management. In this Demo we will not configure Singe Sign on.
This is an example for authorization handling in ArgoCD. We will create a simple user alice with limited access rights

- ArgoCD Config Map Example for Real Projects: https://argo-cd.readthedocs.io/en/stable/operator-manual/argocd-cm-yaml/
- ArgoCD Config Map Example for RBAC: https://argo-cd.readthedocs.io/en/stable/operator-manual/rbac/#tying-it-all-together

## Project Structure

TODO

## 1. getting started from scratch
This guide is foru you if you don't have already installed ArgoCD. If you already have ArgoCD 
you can skip to step 2.

### 1.1 First Fork this Repository and export your github username:
```sh
export USER=<USER>
```

### 1.2 install Argo CD
```sh
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

### 1.3 get init password for UI:
```sh
argocd admin initial-password -n argocd
```

### 1.4 port-forward your ArgoCD UI:
```sh
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
kubectl port-forward svc/argocd-server -n argocd 8080:443
```
### 1.5 argocd CLI
make sure your argocd-cli is connected to your argocd instance. for demonstration purposes you can 
connect the CLI with the instance like this and paste your user and password:
```sh
argocd login 127.0.0.1:8080
```

## 2. Configure ArgoCD

Configure ArgoCD to manage itselfs configurations in Git:
```sh
argocd app create argocd-config \
--repo https://github.com/$USER/argocd-config-demo \
--path config \
--dest-server https://kubernetes.default.svc \
--dest-namespace argocd \
--sync-policy automated
```

Now we should see a new account called **alice**. We can check this with following command:
```sh
$ argocd account list
NAME   ENABLED  CAPABILITIES
admin  true     login
alice  true     apiKey, login
```

Alice currently has no password set. If you logged in with the admin account recently you can set a new password for alice in order to login with her account in the UI:
```sh
# if you are managing users as the admin user, <current-user-password> should be the current admin password.
argocd account update-password \
  --account <name> \
  --current-password <current-user-password> \
  --new-password <new-user-password>
```

## Apps 

Apps using the **App of Apps** pattern

More Details see: https://argo-cd.readthedocs.io/en/stable/operator-manual/cluster-bootstrapping/

### Bootstrap

```sh
export USER=<USER>
argocd app create apps \
    --dest-namespace argocd \
    --dest-server https://kubernetes.default.svc \
    --repo https://github.com/$USER/argocd-config-demo \
    --path apps  
argocd app sync apps  
```

After adding the app sync via CLI
```sh
argocd app sync -l app.kubernetes.io/instance=apps
```

## Results

Now we have two users: **admin** and **alice**. Admin has all access rights including execute shell commands in pods. for that we modified the configmap
```sh
p, role:org-admin, exec, create, *, allow
```


