# Known Issues

- rpc error: code = Unknown desc = Get "...": context deadline exceeded (Client.Timeout exceeded while awaiting headers)
- Namespace gitops-system get stuck in terminating state when deleting ArgoCD.

```
https://vuptime.io/cards/k8s_remove_a_namespace_stuck_in_terminating/
kubectl delete apps <app_name> -n <namespace> did not work for me.
The response application.argoproj.io "<app_name>" deleted looks like it worked, but if I
kubectl get apps -n <namespace>, the app still remains.
I had a finalizer in the app, going in and editing that to - finalizer: [] allowed it to delete.
```

- cert-manager is installed in cert-manager namespace rather than gitops-system namespace because of technical issues.
- ArgoCD CLI may have issues as there is no ingress resource for gRPC.
- After configuring `Pod Templates` in Jenkins, the `Command to run` section will be changed to `sleep`, remove the value and keep it empty for jnlp agents.
- sealed-secrets will not update existings secrets
