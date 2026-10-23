# Demo 1: Hardening a Kubernetes Cluster with Kubescape CLI

This demo follows the [official Kubescape guide](https://kubescape.io/docs/guides/kubescape-cli/) for hardening Kubernetes workloads using the Kubescape command-line tool. Demo shows the basic example of Kubescape scanning, analysis of the derived report and applying remediation patches.

## Prerequisites

- [Docker](https://docs.docker.com/engine/install/)
- [kind](https://kind.sigs.k8s.io/docs/user/quick-start/#installation)
- [kubectl](https://kubernetes.io/docs/tasks/tools/)
- `curl`

## Create a Cluster

The cluster is defined in `kind/config.yaml` — 1 control plane node and 2 workers.

```bash
kind create cluster --config kind/config.yaml
```

kind automatically merges the kubeconfig into `~/.kube/config` and sets the current context.

Verify the cluster is running:

```bash
kubectl get nodes
```

## Install Kubescape CLI

```bash
curl -s https://raw.githubusercontent.com/kubescape/kubescape/master/install.sh | /bin/bash
```

The installer places the binary in `~/.kubescape/bin/`, which is not in PATH by default. Add it:

```bash
echo 'export PATH=$PATH:$HOME/.kubescape/bin' >> ~/.bashrc
source ~/.bashrc
```

Verify the installation:

```bash
kubescape version
```

## Deploy Simple Nginx App

To deploy simple Nginx app:
```bash
kubectl apply -f nginx-app.yml
```

Verify that app pod is running:
```bash
kubectl get pods -n app
```

## Run First Scan with Kubescape CLI

By default Kubescape will scan against the metrics of MITRE and NSA frameworks:
```bash
kubescape scan
```

You should see output similar to this with calculated compliance score:
```bash
Security posture overview for cluster: 'kind-kubescape-cli-demo'

In this overview, Kubescape shows you a summary of your cluster security posture, including the number of users who can perform administrative actions. For each result greater than 0, you should evaluate its need, and then define an exception to allow it. This baseline can be used to detect drift in future.

Control plane
╭────┬─────────────────────────────────────────────┬───────────────────────────────────────────╮
│    │ Control name                                │ Docs                                      │
├────┼─────────────────────────────────────────────┼───────────────────────────────────────────┤
│ ✅ │ API server insecure port is enabled         │ https://kubescape.io/docs/controls/c-0005 │
│ ❌ │ Audit logs enabled                          │ https://kubescape.io/docs/controls/c-0067 │
│ ⚠️ │ Disable anonymous access to Kubelet service │ https://kubescape.io/docs/controls/c-0069 │
│ ⚠️ │ Enforce Kubelet client TLS authentication   │ https://kubescape.io/docs/controls/c-0070 │
│ ❌ │ PSP enabled                                 │ https://kubescape.io/docs/controls/c-0068 │
│ ✅ │ RBAC enabled                                │ https://kubescape.io/docs/controls/c-0088 │
│ ❌ │ Secret/etcd encryption enabled              │ https://kubescape.io/docs/controls/c-0066 │
╰────┴─────────────────────────────────────────────┴───────────────────────────────────────────╯
* This control is scanned exclusively by the Kubescape operator, not the Kubescape CLI. Install the Kubescape operator:
     https://kubescape.io/docs/install-operator/.

Access control
╭────────────────────────────────────────────────────┬───────────┬────────────────────────────────────╮
│ Control name                                       │ Resources │ View details                       │
├────────────────────────────────────────────────────┼───────────┼────────────────────────────────────┤
│ Access Kubernetes dashboard                        │     0     │ $ kubescape scan control C-0014 -v │
│ Access container service account                   │     10    │ $ kubescape scan control C-0053 -v │
│ Administrative Roles                               │     1     │ $ kubescape scan control C-0035 -v │
│ CoreDNS poisoning                                  │     1     │ $ kubescape scan control C-0037 -v │
│ Delete Kubernetes events                           │     1     │ $ kubescape scan control C-0031 -v │
│ List Kubernetes secrets                            │     1     │ $ kubescape scan control C-0015 -v │
│ Minimize access to create pods                     │     2     │ $ kubescape scan control C-0188 -v │
│ Minimize wildcard use in Roles and ClusterRoles    │     1     │ $ kubescape scan control C-0187 -v │
│ Portforwarding privileges                          │     1     │ $ kubescape scan control C-0063 -v │
│ Prevent containers from allowing command execution │     1     │ $ kubescape scan control C-0002 -v │
│ Roles with delete capabilities                     │     3     │ $ kubescape scan control C-0007 -v │
│ Validate admission controller (mutating)           │     0     │ $ kubescape scan control C-0039 -v │
│ Validate admission controller (validating)         │     0     │ $ kubescape scan control C-0036 -v │
╰────────────────────────────────────────────────────┴───────────┴────────────────────────────────────╯

Secrets
╭─────────────────────────────────────────────────┬───────────┬────────────────────────────────────╮
│ Control name                                    │ Resources │ View details                       │
├─────────────────────────────────────────────────┼───────────┼────────────────────────────────────┤
│ Applications credentials in configuration files │     1     │ $ kubescape scan control C-0012 -v │
│ Automatic mapping of service account            │     3     │ $ kubescape scan control C-0034 -v │
╰─────────────────────────────────────────────────┴───────────┴────────────────────────────────────╯

Network
╭─────────────────────────────┬───────────┬────────────────────────────────────╮
│ Control name                │ Resources │ View details                       │
├─────────────────────────────┼───────────┼────────────────────────────────────┤
│ Cluster internal networking │     2     │ $ kubescape scan control C-0054 -v │
│ Container hostPort          │     1     │ $ kubescape scan control C-0044 -v │
│ HostNetwork access          │     1     │ $ kubescape scan control C-0041 -v │
│ Ingress and Egress blocked  │     3     │ $ kubescape scan control C-0030 -v │
│ Missing network policy      │     4     │ $ kubescape scan control C-0260 -v │
╰─────────────────────────────┴───────────┴────────────────────────────────────╯

Workload
╭───────────────────────────────────────────────────────────────────────┬───────────┬────────────────────────────────────╮
│ Control name                                                          │ Resources │ View details                       │
├───────────────────────────────────────────────────────────────────────┼───────────┼────────────────────────────────────┤
│ CVE-2021-25741 - Using symlink for arbitrary host file system access. │     0     │ $ kubescape scan control C-0058 -v │
│ CVE-2021-25742-nginx-ingress-snippet-annotation-vulnerability         │     0     │ $ kubescape scan control C-0059 -v │
│ Exposed sensitive interfaces                                          │     0     │ $ kubescape scan control C-0021 -v │
│ Kubernetes CronJob                                                    │     0     │ $ kubescape scan control C-0026 -v │
│ Mount service principal                                               │     0     │ $ kubescape scan control C-0020 -v │
│ SSH server running inside container                                   │     0     │ $ kubescape scan control C-0042 -v │
╰───────────────────────────────────────────────────────────────────────┴───────────┴────────────────────────────────────╯

Supply chain
╭──────────────────────────────────────────────┬───────────┬────────────────────────────────────╮
│ Control name                                 │ Resources │ View details                       │
├──────────────────────────────────────────────┼───────────┼────────────────────────────────────┤
│ Anonymous access enabled                     │     1     │ $ kubescape scan control C-0262 -v │
│ Authenticated user has sensitive permissions │     0     │ $ kubescape scan control C-0265 -v │
╰──────────────────────────────────────────────┴───────────┴────────────────────────────────────╯

Resource management
╭──────────────────────────────────────┬───────────┬────────────────────────────────────╮
│ Control name                         │ Resources │ View details                       │
├──────────────────────────────────────┼───────────┼────────────────────────────────────┤
│ Ensure CPU limits are set            │     2     │ $ kubescape scan control C-0270 -v │
│ Ensure memory limits are set         │     2     │ $ kubescape scan control C-0271 -v │
│ Nginx Ingress Controller End of Life │     0     │ $ kubescape scan control C-0292 -v │
╰──────────────────────────────────────┴───────────┴────────────────────────────────────╯

Storage
╭─────────────────────────┬───────────┬────────────────────────────────────╮
│ Control name            │ Resources │ View details                       │
├─────────────────────────┼───────────┼────────────────────────────────────┤
│ HostPath mount          │     1     │ $ kubescape scan control C-0048 -v │
│ Writable hostPath mount │     1     │ $ kubescape scan control C-0045 -v │
╰─────────────────────────┴───────────┴────────────────────────────────────╯

Node escape
╭────────────────────────────────┬───────────┬────────────────────────────────────╮
│ Control name                   │ Resources │ View details                       │
├────────────────────────────────┼───────────┼────────────────────────────────────┤
│ Allow privilege escalation     │     3     │ $ kubescape scan control C-0016 -v │
│ Host PID/IPC privileges        │     0     │ $ kubescape scan control C-0038 -v │
│ Immutable container filesystem │     3     │ $ kubescape scan control C-0017 -v │
│ Insecure capabilities          │     1     │ $ kubescape scan control C-0046 -v │
│ Linux hardening                │     3     │ $ kubescape scan control C-0055 -v │
│ Non-root containers            │     3     │ $ kubescape scan control C-0013 -v │
│ Privileged container           │     1     │ $ kubescape scan control C-0057 -v │
╰────────────────────────────────┴───────────┴────────────────────────────────────╯


Highest-stake workloads
───────────────────────

High-stakes workloads are defined as those which Kubescape estimates would have the highest impact if they were to be exploited.

1. namespace: kube-system, name: kindnet, kind: DaemonSet
   '$ kubescape scan workload DaemonSet/kindnet --namespace kube-system'
2. namespace: app, name: nginx-app, kind: Pod
   '$ kubescape scan workload Pod/nginx-app --namespace app'
3. namespace: local-path-storage, name: local-path-provisioner, kind: Deployment
   '$ kubescape scan workload Deployment/local-path-provisioner --namespace local-path-storage'


Compliance Score
────────────────

The compliance score is calculated by multiplying control failures by the number of failures against supported compliance frameworks. Remediate controls, or configure your cluster baseline with exceptions, to improve this score.

* MITRE: 76.90%
* NSA: 69.18%

View a full compliance report by running '$ kubescape scan framework nsa' or '$ kubescape scan framework mitre'
```

## Scan Specific Finding

In this example scan against the specific finding "Missing Network Policy" from the output:
```bash
kubescape scan control C-0260 -v
```

You should see output similar to this:
```bash
################################################################################
ApiVersion: apps/v1
Kind: Deployment
Name: local-path-provisioner
Namespace: local-path-storage

Controls: 1 (Failed: 1, action required: 0)

╭──────────┬────────────────────────┬────────────────────────────────────┬──────────────────────╮
│ Severity │ Control name           │ Docs                               │ Assisted remediation │
├──────────┼────────────────────────┼────────────────────────────────────┼──────────────────────┤
│ Medium   │ Missing network policy │ https://hub.armosec.io/docs/c-0260 │                      │
╰──────────┴────────────────────────┴────────────────────────────────────┴──────────────────────╯

################################################################################
ApiVersion: v1
Kind: Pod
Name: nginx-app
Namespace: app

Controls: 1 (Failed: 1, action required: 0)

╭──────────┬────────────────────────┬────────────────────────────────────┬──────────────────────╮
│ Severity │ Control name           │ Docs                               │ Assisted remediation │
├──────────┼────────────────────────┼────────────────────────────────────┼──────────────────────┤
│ Medium   │ Missing network policy │ https://hub.armosec.io/docs/c-0260 │                      │
╰──────────┴────────────────────────┴────────────────────────────────────┴──────────────────────╯

################################################################################
ApiVersion: apps/v1
Kind: DaemonSet
Name: kindnet
Namespace: kube-system

Controls: 1 (Failed: 1, action required: 0)

╭──────────┬────────────────────────┬────────────────────────────────────┬──────────────────────╮
│ Severity │ Control name           │ Docs                               │ Assisted remediation │
├──────────┼────────────────────────┼────────────────────────────────────┼──────────────────────┤
│ Medium   │ Missing network policy │ https://hub.armosec.io/docs/c-0260 │                      │
╰──────────┴────────────────────────┴────────────────────────────────────┴──────────────────────╯

################################################################################
ApiVersion: v1
Kind: Pod
Name: kube-apiserver-kubescape-cli-demo-control-plane
Namespace: kube-system

Controls: 1 (Failed: 1, action required: 0)

╭──────────┬────────────────────────┬────────────────────────────────────┬──────────────────────╮
│ Severity │ Control name           │ Docs                               │ Assisted remediation │
├──────────┼────────────────────────┼────────────────────────────────────┼──────────────────────┤
│ Medium   │ Missing network policy │ https://hub.armosec.io/docs/c-0260 │                      │
╰──────────┴────────────────────────┴────────────────────────────────────┴──────────────────────╯


╭─────────────────┬───╮
│        Controls │ 1 │
│          Passed │ 0 │
│          Failed │ 1 │
│ Action Required │ 0 │
╰─────────────────┴───╯

Failed resources by severity:

╭──────────┬───╮
│ Critical │ 0 │
│     High │ 0 │
│   Medium │ 4 │
│      Low │ 0 │
╰──────────┴───╯

╭──────────┬────────────────────────┬──────────────────┬───────────────┬──────────────────╮
│ Severity │ Control name           │ Failed resources │ All Resources │ Compliance score │
├──────────┼────────────────────────┼──────────────────┼───────────────┼──────────────────┤
│  Medium  │ Missing network policy │         4        │       23      │        83%       │
├──────────┼────────────────────────┼──────────────────┼───────────────┼──────────────────┤
│          │    Resource Summary    │         4        │       23      │      82.61%      │
╰──────────┴────────────────────────┴──────────────────┴───────────────┴──────────────────╯
```

## Remediate a Network Control Failure

To remediate our finding apply the network policy fix to the cluster:
```bash
kubectl apply -f netpol.yml
```

Then re-scan to verify that warning is gone and remediation was applied:
```bash
kubescape scan control C-0260 -v
```

You should see the output:
```bash
 ✅  Done scanning. Cluster: kind-kubescape-cli-demo
 ✅  Done aggregating results


──────────────────────────────────────────────────



╭─────────────────┬───╮
│        Controls │ 1 │
│          Passed │ 1 │
│          Failed │ 0 │
│ Action Required │ 0 │
╰─────────────────┴───╯

Failed resources by severity:

╭──────────┬───╮
│ Critical │ 0 │
│     High │ 0 │
│   Medium │ 0 │
│      Low │ 0 │
╰──────────┴───╯

╭──────────┬────────────────────────┬──────────────────┬───────────────┬──────────────────╮
│ Severity │ Control name           │ Failed resources │ All Resources │ Compliance score │
├──────────┼────────────────────────┼──────────────────┼───────────────┼──────────────────┤
│  Medium  │ Missing network policy │         0        │       28      │       100%       │
├──────────┼────────────────────────┼──────────────────┼───────────────┼──────────────────┤
│          │    Resource Summary    │         0        │       28      │      100.00%     │
╰──────────┴────────────────────────┴──────────────────┴───────────────┴──────────────────╯
```

## Cleanup

```bash
kind delete cluster --name kubescape-cli-demo
```


## References
- [https://github.com/kubescape/regolibrary](https://github.com/kubescape/regolibrary)