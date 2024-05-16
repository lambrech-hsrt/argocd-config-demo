# argocd-config-demo
Argo CD Config Demo Repository with user management and RBAC management. In this Demo we will not configure Singe Sign on.
This is an example for authorization handling in ArgoCD. We will create a simple user alice with limited access rights

This example also includes a simple demonstration for Diffing with ArgoCD (replicas).

More Details for RBAC and the ArgoCD Config Map:
- ArgoCD Config Map Example for Real Projects: https://argo-cd.readthedocs.io/en/stable/operator-manual/argocd-cm-yaml/
- ArgoCD Config Map Example for RBAC: https://argo-cd.readthedocs.io/en/stable/operator-manual/rbac/#tying-it-all-together

## Project Structure

```
.
├── apps/
│   ├── templates/
│   │   ├── helm-guestbook.yaml
│   │   ├── helm-hooks.yaml
│   │   ├── kustomize-guestbook.yaml
│   │   └── ...
│   ├── Chart.yaml
│   └── values.yaml
└── config/
    ├── argocd-cm.yaml --> Argo CD Config Map (for Users and other Stuff)
    └── argocd-rbac-cm.yaml --> ArgoCD Role Based Access Controll Configuration
```

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

Wait for all pods to be ready. this can take a few minutes.

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

### 2.1 Add ArgoCD Config to Git and check user
Configure ArgoCD to manage itselfs configurations in Git:
```sh
argocd app create argocd-config \
--repo https://github.com/$USER/argocd-config-demo \
--path config \
--dest-server https://kubernetes.default.svc \
--dest-namespace argocd \
--sync-policy automated
```
<img src="https://github.com/lambrech-hsrt/argocd-config-demo/blob/main/.img/argo-config.png" alt="ressources">

You can now login with the admin in the UI and you will the our new config application containing the RBAC
and the argocd configmap

Also we should see a new account called **alice**. We can check this with following command:
```sh
$ argocd account list
NAME   ENABLED  CAPABILITIES
admin  true     login
alice  true     apiKey, login
```

### 2.2 new password for Alice
Alice currently has no password set. If you logged in with the admin account recently you can set a new password for alice in order to login with her account in the UI:
```sh
# if you are managing users as the admin user, <current-user-password> should be the current admin password.
argocd account update-password \
  --account <name> \
  --current-password <current-user-password> \
  --new-password <new-user-password>
```

## 3. Adding Demo Apps 

In this Section we are going to add some sample apps to test our permission levels

The Apps are using the **App of Apps** pattern from ArgoCD details here: https://argo-cd.readthedocs.io/en/stable/operator-manual/cluster-bootstrapping/

### 3.1 Bootstrap

We can bootstrap with one single command our demo app:

```sh
export USER=<USER>
argocd app create apps \
    --dest-namespace argocd \
    --dest-server https://kubernetes.default.svc \
    --repo https://github.com/$USER/argocd-config-demo \
    --path apps  
```

By default the applications nested in ``apps`` are not sync yet we cann sync them with the following command or with
the UI
```sh
argocd app sync -l app.kubernetes.io/instance=apps
```

## Results

Now we have two users: **admin** and **alice**. Admin has all access rights including execute shell commands in pods. for that we modified the configmap. 

Alice only can view the applications and not view the configuration app. If you try to change anything with alice you will get an ``permission denied`` error message in the UI.


