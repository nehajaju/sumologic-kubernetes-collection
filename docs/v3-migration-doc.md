# Kubernetes Collection `v3.0.0` - Breaking Changes

- [Changes](#changes)
- [How to upgrade](#how-to-upgrade)
  - [Requirements](#requirements)
  - [Manual steps](#manual-steps)
    - [Upgrade kube-prometheus-stack](#upgrade-kube-prometheus-stack)
  - [Replace special configuration values marked by 'replace' suffix](#replace-special-configuration-values-marked-by-replace-suffix)
    - [Otelcol StatefulSets](#otelcol-statefulsets)

Based on the feedback from our users, we will be introducing several changes
to the Sumo Logic Kubernetes Collection solution.

In this document we detail the changes as well as the exact steps for migration.

## Changes

- Upgrading kube-prometheus stack

  We are updating Kube-prometheus-stack to newest available version.
  Major feature related to that change is upgrading kube-state-metrics to v2

- Removing mechanism to replace values in configuration for traces marked by 'replace' suffix
- Moving direct configuration of OpenTelemetry Collector for log metadata

  Removed explicit configuration for otelcol under `metadata.logs.config`.
  Added option to merge configuration under `metadata.logs.config.merge`
  or overwrite default configuration `metadata.logs.config.override`
- Moving direct configuration of OpenTelemetry Collector for metrics metadata

  Removed explicit configuration for otelcol under `metadata.metrics.config`.
  Added option to merge configuration under `metadata.metrics.config.merge`
  or overwrite default configuration `metadata.metrics.config.override`
- Removing support for `sumologic.cluster.load_config_file`.
  Leaving this configuration will result in setup job failure.
- Upgrading Falco helm chart to `v2.4.2` which changed their configuration:
  Please validate and adjust your configuration to new version according to [Falco documentation]

- Moving parameters from `fluentd.logs.containers` to `sumologic.logs.container`
  - moved `fluentd.logs.containers.sourceHost` to `sumologic.logs.container.sourceHost`
  - moved `fluentd.logs.containers.sourceName` to `sumologic.logs.container.sourceName`
  - moved `fluentd.logs.contianers.sourceCategory` to `sumologic.logs.container.sourceCategory`
  - moved  `fluentd.logs.containers.sourceCategoryPrefix` to `sumologic.logs.container.sourceCategoryPrefix`
  - moved  `fluentd.logs.contianers.sourceCategoryReplaceDash` to `sumologic.logs.container.sourceCategoryReplaceDash`
  - moved `fluentd.logs.containers.excludeContainerRegex` to `sumologic.logs.container.excludeContainerRegex`
  - moved `fluentd.logs.containers.excludeHostRegex` to `sumologic.logs.container.excludeHostRegex`
  - moved `fluentd.logs.containers.excludeNamespaceRegex` to `sumologic.logs.container.excludeNamespaceRegex`
  - moved `fluentd.logs.containers.excludePodRegex` to `sumologic.logs.container.excludePodRegex`
  - moved `fluentd.logs.containers.sourceHost` to `sumologic.logs.container.sourceHost`
  - moved `fluentd.logs.containers.perContainerAnnotationsEnabled` to `sumologic.logs.container.perContainerAnnotationsEnabled`
  - moved `fluentd.logs.containers.perContainerAnnotationPrefixes` to `sumologic.logs.container.perContainerAnnotationPrefixes`

- Moving parameters from `fluentd.logs.kubelet` to `sumologic.logs.kubelet`
  - moved `fluentd.logs.kubelet.sourceName` to `sumologic.logs.kubelet.sourceName`
  - moved `fluentd.logs.kubelet.sourceCategory` to `sumologic.logs.kubelet.sourceCategory`
  - moved `fluentd.logs.kubelet.sourceCategoryPrefix` to `sumologic.logs.kubelet.sourceCategoryPrefix`
  - moved `fluentd.logs.kubelet.sourceCategoryReplaceDash` to `sumologic.logs.kubelet.sourceCategoryReplaceDash`
  - moved `fluentd.logs.kubelet.excludeFacilityRegex` to `sumologic.logs.kubelet.excludeFacilityRegex`
  - moved `fluentd.logs.kubelet.excludeHostRegex` to `sumologic.logs.kubelet.excludeHostRegex`
  - moved `fluentd.logs.kubelet.excludePriorityRegex` to `sumologic.logs.kubelet.excludePriorityRegex`
  - moved `fluentd.logs.kubelet.excludeUnitRegex` to `sumologic.logs.kubelet.excludeUnitRegex`

- Moving parameters from `fluentd.logs.systemd` to `sumologic.logs.systemd`
  - moved `fluentd.logs.systemd.sourceName` to `sumologic.logs.systemd.sourceName`
  - moved `fluentd.logs.systemd.sourceCategory` to `sumologic.logs.systemd.sourceCategory`
  - moved `fluentd.logs.systemd.sourceCategoryPrefix` to `sumologic.logs.systemd.sourceCategoryPrefix`
  - moved `fluentd.logs.systemd.sourceCategoryReplaceDash` to `sumologic.logs.systemd.sourceCategoryReplaceDash`
  - moved `fluentd.logs.systemd.excludeFacilityRegex` to `sumologic.logs.systemd.excludeFacilityRegex`
  - moved `fluentd.logs.systemd.excludeHostRegex` to `sumologic.logs.systemd.excludeHostRegex`
  - moved `fluentd.logs.systemd.excludePriorityRegex` to `sumologic.logs.systemd.excludePriorityRegex`
  - moved `fluentd.logs.systemd.excludeUnitRegex` to `sumologic.logs.systemd.excludeUnitRegex`

- Moving parameters from `fluentd.logs.default` to `sumologic.logs.defaultFluentd`
  - moved `fluentd.logs.default.sourceName` to `sumologic.logs.defaultFluentd.sourceName`
  - moved `fluentd.logs.default.sourceCategory` to `sumologic.logs.defaultFluentd.sourceCategory`
  - moved `fluentd.logs.default.sourceCategoryPrefix` to `sumologic.logs.defaultFluentd.sourceCategoryPrefix`
  - moved `fluentd.logs.default.sourceCategoryReplaceDash` to `sumologic.logs.defaultFluentd.sourceCategoryReplaceDash`
  - moved `fluentd.logs.default.excludeFacilityRegex` to `sumologic.logs.defaultFluentd.excludeFacilityRegex`
  - moved `fluentd.logs.default.excludeHostRegex` to `sumologic.logs.defaultFluentd.excludeHostRegex`
  - moved `fluentd.logs.default.excludePriorityRegex` to `sumologic.logs.defaultFluentd.excludePriorityRegex`
  - moved `fluentd.logs.default.excludeUnitRegex` to `sumologic.logs.defaultFluentd.excludeUnitRegex`

- Upgrading Metrics Server to `6.2.4`. In case of changing `metrics-server.*` configuration
  please see [upgrading section of chart's documentation][metrics-server-upgrade].

- Upgrading Tailing Sidecar Operator helm chart to v0.5.5. There is no breaking change if using annotations only.

## How to upgrade

### Requirements

- `helm3`
- `kubectl`
- `jq`

### Manual steps

1. Perform required manual steps:
    - [Upgrade kube-prometheus-stack](#upgrade-kube-prometheus-stack)
2. Delete the following StatefulSets (otelcol):
    - [Otelcol StatefulSets](#otelcol-statefulsets)

#### Upgrade kube-prometheus-stack

Upgrade of kube-prometheus-stack is a breaking change and requires manual steps:

- Upgrading prometheus CRDs:

  ```bash
  kubectl apply --server-side --force-conflicts -f https://raw.githubusercontent.com/prometheus-operator/prometheus-operator/v0.58.0/example/prometheus-operator-crd/monitoring.coreos.com_alertmanagerconfigs.yaml
  kubectl apply --server-side --force-conflicts -f https://raw.githubusercontent.com/prometheus-operator/prometheus-operator/v0.58.0/example/prometheus-operator-crd/monitoring.coreos.com_alertmanagers.yaml
  kubectl apply --server-side --force-conflicts -f https://raw.githubusercontent.com/prometheus-operator/prometheus-operator/v0.58.0/example/prometheus-operator-crd/monitoring.coreos.com_podmonitors.yaml
  kubectl apply --server-side --force-conflicts -f https://raw.githubusercontent.com/prometheus-operator/prometheus-operator/v0.58.0/example/prometheus-operator-crd/monitoring.coreos.com_probes.yaml
  kubectl apply --server-side --force-conflicts -f https://raw.githubusercontent.com/prometheus-operator/prometheus-operator/v0.58.0/example/prometheus-operator-crd/monitoring.coreos.com_prometheuses.yaml
  kubectl apply --server-side --force-conflicts -f https://raw.githubusercontent.com/prometheus-operator/prometheus-operator/v0.58.0/example/prometheus-operator-crd/monitoring.coreos.com_prometheusrules.yaml
  kubectl apply --server-side --force-conflicts -f https://raw.githubusercontent.com/prometheus-operator/prometheus-operator/v0.58.0/example/prometheus-operator-crd/monitoring.coreos.com_servicemonitors.yaml
  kubectl apply --server-side --force-conflicts -f https://raw.githubusercontent.com/prometheus-operator/prometheus-operator/v0.58.0/example/prometheus-operator-crd/monitoring.coreos.com_thanosrulers.yaml
  ```

  due to:

  ```text
  Error: UPGRADE FAILED: error validating "": error validating data: ValidationError(Prometheus.spec): unknown field "shards" in com.coreos.monitoring.v1.Prometheus.spec
  ```

- Patching `kube-state-metrics` deployment:

  ```bash
  kubectl get deployment \
    --namespace="${NAMESPACE}" \
    --selector 'app.kubernetes.io/name=kube-state-metrics' \
    -o json | \
  jq ". | .items[].spec.selector.matchLabels[\"app.kubernetes.io/instance\"] |= \"${HELM_RELEASE_NAME}\"" | \
  kubectl apply \
    --namespace="${NAMESPACE}" \
    --force \
    --filename -
  ```

  due to:

  ```text
  Error: UPGRADE FAILED: cannot patch "collection-kube-state-metrics" with kind Deployment: Deployment.apps "collection-kube-state-metrics" is invalid: spec.selector: Invalid value: v1.LabelSelector{MatchLabels:map[string]string{"app.kubernetes.io/instance":"collection", "app.kubernetes.io/name":"kube-state-metrics"}, MatchExpressions:[]v1.LabelSelectorRequirement(nil)}: field is immutable
  ```

- In case of overriding any of the `repository` property under the `kube-prometheus-stack` property,
  please follow the `kube-prometheus-stack` [migration doc][kube-prometheus-stack-image-migration] on that.

### Replace special configuration values marked by 'replace' suffix

Mechanism to replace special configuration values for traces marked by 'replace' suffix was removed and following special values in configuration are no longer automatically replaced and they need to be changed:

- `exporters.otlptraces.endpoint.replace`
- `exporters.otlpmetrics.endpoint.replace`
- `processors.source.collector.replace`
- `processors.source.name.replace`
- `processors.source.category.replace`
- `processors.source.category_prefix.replace`
- `processors.source.category_replace_dash.replace`
- `processors.source.exclude_namespace_regex.replace`
- `processors.source.exclude_pod_regex.replace`
- `processors.source.exclude_container_regex.replace`
- `processors.source.exclude_host_regex.replace`
- `processors.resource.cluster.replace`
- `exporters.sumologic.source_name.replace`
- `exporters.sumologic.source_category.replace`

Above special configuration values can be replaced either to direct values or be set as reference to other parameters form `values.yaml`.

#### Otelcol StatefulSets

If you're using `otelcol` as the logs/metrics metadata provider, please run one or both of the following commands to manually delete StatefulSets in helm chart v2 before upgrade:

  ```
  kubectl delete sts --namespace=my-namespace --cascade=false my-release-sumologic-otelcol-logs
  kubectl delete sts --namespace=my-namespace --cascade=false my-release-sumologic-otelcol-metrics
  ```

### Known issues

#### Cannot delete pod if using Tailing Sidecar Operator

If you are using Tailing Sidecar Operator and see the following error:

```
Error from server: admission webhook "tailing-sidecar.sumologic.com" denied the request: there is no content to decode
```

Please try to remove pod later.

[Falco documentation]: https://github.com/falcosecurity/charts/tree/falco-2.4.2/falco
[metrics-server-upgrade]: https://github.com/bitnami/charts/tree/5b09f7a7c0d9232f5752840b6c4e5cdc56d7f796/bitnami/metrics-server#to-600
[kube-prometheus-stack-image-migration]: https://github.com/prometheus-community/helm-charts/tree/kube-prometheus-stack-42.1.0/charts/kube-prometheus-stack#from-41x-to-42x
