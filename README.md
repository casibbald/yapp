## yapp-rs
[![ci](https://github.com/casibbald/yapp/actions/workflows/ci.yml/badge.svg)](https://github.com/casibbald/yapp/actions/workflows/ci.yml)

[//]: # ([![docker image]&#40;https://img.shields.io/docker/pulls/clux/controller.svg&#41;]&#40;)
[//]: # (https://hub.docker.com/r/clux/controller/tags/&#41;)

A rust kubernetes reference controller for a [`Document` resource](https://github.com/casibbald/yapp/blob/main/yaml/crd.yaml) using [kube](https://github.com/kube-rs/kube/), with observability instrumentation.

The `Controller` object reconciles `Document` instances when changes to it are detected, writes to its `.status` object, creates associated events, and uses finalizers for guaranteed delete handling.

## Installation

### CRD
Apply the CRD from [cached file](yaml/crd.yaml), or pipe it from `crdgen` to pickup schema changes:

```sh
cargo run --bin crdgen | kubectl apply -f -
```

### Controller

Install the controller via `helm` by setting your preferred settings. For defaults:

```sh
helm template charts/doc-controller | kubectl apply -f -
kubectl wait --for=condition=available deploy/doc-controller --timeout=30s
kubectl port-forward service/doc-controller 8080:80
```

### Opentelemetry

Build and run with `telemetry` feature, or configure it via `helm`:

```sh
helm template charts/doc-controller --set tracing.enabled=true | kubectl apply -f -
```

This requires an opentelemetry collector in your cluster. [Tempo](https://github.com/grafana/helm-charts/tree/main/charts/tempo) / [opentelemetry-operator](https://github.com/open-telemetry/opentelemetry-helm-charts/tree/main/charts/opentelemetry-operator) / [grafana agent](https://github.com/grafana/helm-charts/tree/main/charts/agent-operator) should all work out of the box. If your collector does not support grpc otlp you need to change the exporter in [`telemetry.rs`](./src/telemetry.rs).

Note that the [images are pushed either with or without the telemetry feature](https://hub.docker.com/r/clux/controller/tags/) depending on whether the tag includes `otel`.

### Metrics

Metrics is available on `/metrics` and a `ServiceMonitor` is configurable from the chart:

```sh
helm template charts/doc-controller --set serviceMonitor.enabled=true | kubectl apply -f -
```

## Running

### Locally

```sh
cargo run
```

or, with optional telemetry:

```sh
OPENTELEMETRY_ENDPOINT_URL=https://0.0.0.0:4317 RUST_LOG=info,kube=trace,controller=debug cargo run --features=telemetry
```

### In-cluster
For prebuilt, edit the [chart values](./charts/yapp-controller/values.yaml) or [snapshotted yaml](./yaml/deployment.yaml) and apply as you see fit (like above).

To develop by building and deploying the image quickly, we recommend using [tilt](https://tilt.dev/), via `tilt up` instead.


