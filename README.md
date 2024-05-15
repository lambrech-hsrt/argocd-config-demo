# argocd-config-demo
Argo CD Config Demo Repository with user management and RBAC management. In this Demo we will not configure Singe Sign on.
This is an example for authorization handling in ArgoCD. We will create a simple user alice with limited access rights

- ArgoCD Config Map Example for Real Projects: https://argo-cd.readthedocs.io/en/stable/operator-manual/argocd-cm-yaml/
- ArgoCD Config Map Example for RBAC: https://argo-cd.readthedocs.io/en/stable/operator-manual/rbac/#tying-it-all-together

## Project Structure

TODO

## vorbereitung
First Fork this Repository and export your github username:
```sh
export USER=<USER>
```

get init password for UI:
```sh
argocd admin initial-password -n argocd
```

If not already done port-forward your ArgoCD UI:
```sh
kubectl port-forward -n argocd svc/argocd-server 8080:80
```

If not already done make sure your argocd-cli is connected to your argocd instance. for demonstration purposes you can 
connect the CLI with the instance like this and paste your user and password:
```sh
argocd login 127.0.0.1:8080
```

## Configure ArgoCD

Configure ArgoCD to manage itselfs configurations in Git:
```sh
argocd app create argocd-config \
--repo https://github.com/$USER/argocd-config-demo \
--path config \
--dest-server https://kubernetes.default.svc \
--dest-namespace argocd \
--sync-policy automated
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


