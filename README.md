# argocd-sample-application

A minimal GitOps proof-of-concept: a small Helm chart (`sample-app`) deployed and
managed by **Argo CD**. Change a value, push to Git, and watch Argo CD reconcile it.

```
argocd-helm-poc/
├── apps/
│   └── sample-app.yaml          # Argo CD Application (the "app of one")
└── charts/
    └── sample-app/              # The Helm chart Argo CD deploys
        ├── Chart.yaml
        ├── values.yaml          # ← edit these knobs, commit, watch the sync
        └── templates/
```

## 1. Push this to your GitHub

From inside this folder:

```bash
git init -b main
git add .
git commit -m "Initial commit: sample-app Helm chart + Argo CD Application"

# Create an empty repo named argocd-helm-poc on GitHub first (no README),
# then point this at it:
git remote add origin https://github.com/<your-username>/argocd-helm-poc.git
git push -u origin main
```

## 2. Wire up the Application

Edit `apps/sample-app.yaml` and replace `<your-username>` in `repoURL`, then apply it
to the cluster where Argo CD is running:

```bash
kubectl apply -f apps/sample-app.yaml
```

(If your repo is private, register credentials first with
`argocd repo add https://github.com/<your-username>/argocd-helm-poc.git --username <user> --password <token>`.)

## 3. Watch it sync

In the Argo CD UI you'll see the `sample-app` Application appear and sync automatically
(auto-sync is on). To view the app itself:

```bash
kubectl port-forward -n sample-app svc/sample-app-sample-app 8081:80
# open http://localhost:8081
```

## 4. Try the GitOps loop

1. Edit `charts/sample-app/values.yaml` — change `message` or bump `replicaCount`.
2. `git commit -am "tweak message" && git push`
3. Watch Argo CD detect the diff and sync it (or click **Refresh** to skip the poll wait).

## 5. Try self-heal

`syncPolicy.automated.selfHeal: true` means manual drift gets reverted. Test it:

```bash
kubectl scale deployment -n sample-app sample-app-sample-app --replicas=5
```

Argo CD will scale it back to whatever Git says. That's the whole point. ✅
