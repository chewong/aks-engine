# Conformant API Model

The default cluster created by aks-engine might not be conformant [by definition](https://github.com/cncf/k8s-conformance#certified-kubernetes). Additional configurations are needed and this example folder contains API models that are conformant per Kubernetes release.

## Usage

To create a conformant aks-engine cluster:

```bash
az login
az account -s ${SUBSCRIPTION_ID}
aks-engine deploy \
  --api-model example/conformance/linux-${K8S_RELEASE}.json \
  --dns-prefix ${DNS_PREFIX} \
  --location ${LOCATION}
```

To kick off a conformance test run, see https://github.com/cncf/k8s-conformance/blob/master/instructions.md#running.

## Conformant Configurations

### `.property.linuxProfile.runUnattendedUpgradesOnBootstrap: false`

- In some scenarios, the conformance test suite performs a health check before running it. With `runUnattendedUpgradesOnBootstrap` enabled, the health check might be performed during an unattended node upgrade (which requires a restart), which causes the health check/test suite to terminate prematurely.

### Releases

The following table describes specific configurations needed to create a conformant aks-engine cluster for each Kubernetes release:

| Kubernetes Release | Additional Configurations                                                                                         |
|--------------------|------------------------------------------------------------------------------------------------------------------|
| 1.17               | N/A                                                                                                              |
| 1.18               | N/A                                                                                                              |
| 1.19               | N/A                                                                                                              |
| 1.20               | `.properties.orchestratorProfile.kubernetesConfig.kubeletConfig["--feature-gates"]: "ExecProbeTimeout=true"` [1]    |
| 1.21               | `.properties.orchestratorProfile.kubernetesConfig.kubeletConfig["--feature-gates"]: "ExecProbeTimeout=true"` [1]    |

[1]: [`ExecProbeTimeout`](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/#configure-probes) is a feature flag that corrects the container probe timeout behavior since it was not respected prior to 1.20. This feature flag is enabled by default but to prevent users from experiencing unexpected behaviors after an upgrade, we have disabled it in https://github.com/Azure/aks-engine/pull/4085. We would have to explicity enable it in our conformant API model so [the container probe conformance test case](https://github.com/kubernetes/kubernetes/blob/9a8da9ee43ec4c20f6c0c549b8172b0c363d89cb/test/e2e/common/container_probe.go#L216) can pass.
