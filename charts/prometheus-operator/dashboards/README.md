# Perses dashboards

One file per dashboard in this directory: `dashboards/<name>.yaml`, where `<name>` must match
the dashboard's `metadata.name`. `templates/perses-dashboards.yaml` turns each file into a
`perses-dashboard-<name>` ConfigMap labeled `perses.dev/resource: "true"`, the perses sidecar
copies it to `/etc/perses/provisioning/`, and perses applies it.

## UI is a scratchpad, git is the source of truth

Perses only loads provisioning **into** its file DB; it never exports back. That means
**edits you save in the UI are overwritten by the committed file**:

- perses applies provisioning at startup and every `config.provisioning.interval` (10m),
- and it overwrites whatever is in its DB with the on-disk version.

So a UI edit survives at most ~10 minutes (or until the next pod restart).

Workflow:

```
1. Open the Perses UI, build/tweak your dashboard, save it. Iterate freely.

2. When happy, export it before the 10m reset:
     percli get dashboard <name> --project odg -o yaml > dashboards/<name>.yaml

   (or use the UI's YAML download button)

3. Commit + `helm template | kubectl apply`.
   Perses picks new/changed files up on the next interval; to apply immediately,
   restart the pod:
     kubectl -n <ns> delete pod -l app.kubernetes.io/name=perses-app
```

Kind names are the perses API kinds (`Project`, `Datasource`, `Dashboard`) — no `Perses`
prefix; that style belongs to the separate perses-operator CRDs, which we don't run.

The `odg` project and the `prometheus` datasource are provisioned from
`templates/perses.yaml` (ConfigMap `perses-datasource-prometheus`), pointing at
`http://prometheus-operated:9090`.

If you ever want the UI to be the durable editor instead, disable provisioning
(`config.provisioning.folders: []` in values) and keep the PVC as the source of truth —
but git then no longer drives dashboards.
