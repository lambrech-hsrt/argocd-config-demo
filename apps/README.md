Apps using the App of Apps pattern

More Details see: https://argo-cd.readthedocs.io/en/stable/operator-manual/cluster-bootstrapping/

## Bootstrap

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