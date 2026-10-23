# prometheus-operator chart

Umbrella chart for the ODG observability stack:

- **prometheus-operator** (templated here, CRDs under `crds/`)
- **kube-state-metrics** (vendored subchart, `charts/kube-state-metrics-*.tgz`)
- **perses** (vendored subchart, `charts/perses-*.tgz`, aliased `perses-app`) — dashboards UI

Perses is installed without its operator: it has no CRDs; dashboards and datasources are
plain ConfigMaps labeled `perses.dev/resource: "true"` in the release namespace, which the
perses sidecar copies into its provisioning folder. See [`dashboards/README.md`](dashboards/README.md)
for the dashboard workflow (UI as scratchpad, git as source of truth).

## Bumping vendored subcharts

```sh
cd charts/prometheus-operator
rm charts/<subchart>-*.tgz
helm pull <repo>/<subchart> --version <version> -d charts/
# update the version in Chart.yaml, then:
helm dependency update .
```

(see the comment block in `Chart.yaml` for the exact repo coordinates per subchart)
