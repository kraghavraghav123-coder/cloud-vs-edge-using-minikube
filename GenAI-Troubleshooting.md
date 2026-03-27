## Generative AI Troubleshooting & Usage Log

**Overview:** Generative AI was used as a technical troubleshooting aid to deploy, network, and telemetry related problems in the Minikube ( Cloud ) environment and MicroK8s ( Edge ) environment. The line of critical roadblocks as well as the AI-suggested solutions is presented below.

### ARM64/AMD64 Image Architecture Mismatch
Whenever pod deployment was attempted during the initial configuration of the Edge VM on an Apple Silicon (M-series) Mac, it failed with an error relating to the format of the executable in the logs. Generative AI confirmed this to be an architecture mismatch; the host system was using ARM64 architecture but Kubernetes was trying to fetch default AMD64 container images. The fix was to clearly state ARM64-compatible images of Alpine and Nginx in the deployment manifests and the pods to go into a successful spin-up in the native mode.

### Calico Node Unauthorized Error
During the process of configurating the Kubernetes network plugin on the Edge node, an error of Calico Node Unauthorized was generated which completely ended communication between pods. Generative AI was able to determine that Calico had been corrupted or configured in an incorrect way to run the version of MicroK8s in use. The assisted solution involved fully removing the current Calico resources of the cluster and appropriately reimplementing the Tigera operator and custom resources definitions based on the environment.

### Prometheus Addon Failure
In trying to install advanced telemetry on the Cloud VM (Minikube), the default minikube addons allow prometheus command to fail. Generative AI has stated that this native, 1-click addition has been debuilt in contemporary Minikube versions as its resource bloat often leads to a VM crash. However, rather than using persistent storage or alerting, AI used customized, lightweight Helm commands to configure a dedicated Prometheus and Grafana stack, deliberately turned off, so as to not violate the strict RAM limits of the VM.

### Missing Dashboard Data
Following the enabling of the native Kubernetes dashboards in an effort to conserve memory the UI showed the Waiting for data error rather than showing the visual graphs which were the anticipated outcome. The metrics-server pipeline was known to be lying, which was identified by generative AI. Manually enabling the metrics-server addon and running the kubectl top pods command to confirm that the data stream was active, then opening the UI, then running the kubectl top pods command again to ensure the data stream continued, was the solution. This saw to it that the dashboards were populated with live CPU and Memory telemetry.

### Edge Dashboard Access Denied
When trying to access the MicroK8s dashboard through normal port-forwarding, a recurring error of "Not Found" was obtained. Generative AI found out that current versions of MicroK8s would lock the dashboard behind a Kong API proxy in a dedicated namespace, which is not normally the case in Minikube deployments. The search on the cluster services where the exact kubernetes-dashboard-kong-proxy service was identified and a secure 0.0.0.0 network bridge was built, and the host Mac browser could now browse the dashboard without difficulties.
