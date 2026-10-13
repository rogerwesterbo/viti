---
theme: default
colorSchema: dark
title: Vitistack
info: |
  ## Vitistack
  Infrastructure Platform for Modern Kubernetes — vitistack.io
class: text-center
transition: slide-left
mdc: true
lineNumbers: false
---

<div class="flex items-center justify-center gap-8 mb-6">
  <img src="/viti-logo.svg" class="h-36" />
  <!-- <span class="text-4xl opacity-60">+</span>
  <img src="/k8s-logo.svg" class="h-32" /> -->
</div>

# Vitistack

<!-- Infrastructure Platform for Modern Kubernetes <K8sLogo /> -->

<div class="pt-8 opacity-80">
🚀 Kubernetes clusters, on multiple machine and kubernetes providers
</div>

<div class="abs-br m-6 text-sm opacity-60">
🌐 vitistack.io · 📖 docs.vitistack.io · 🐙 github.com/vitistack
</div>

<!--
Overview: who I am, what we'll cover today.
- Why was Vitistack built
- What Vitistack is
- Operators and the control loop
- Creating a cluster: NetworkNamespace + KubernetesCluster
- Demo / diagram, experiences, future
- Questions
-->

---

# What is Vitistack? 🤔

<div class="grid grid-cols-[3fr_2fr] gap-10 mt-4">
<div>

<v-clicks>

- 🏗️ An **infrastructure platform** for running Kubernetes with many machine-providers and kubernetes-providers
- 📜 Kubernetes clusters as **declarative resources** (CRDs)
- 📦 General objects describing the environment and config of a platform
- 🤖 Built from **operators** — each does one job well
- 🔓 Open source building blocks
- 🎯 Goals: cost-efficient, flexible, secure, interoperability, customizable, see what happens in the "magic" box

</v-clicks>

</div>
<div>

<v-click>

### 🧱 Building blocks

| _Area_                 | _Examples_                                   |
| ---------------------- | -------------------------------------------- |
| <K8sLogo /> Kubernetes | **Talos**, **AKS**, K0s, Scaleway,etc        |
| 🖥️ Machines            | **KubeVirt**, **Proxmox**, (future Physical) |
| 🌐 Network / IP        | **Static IP**, **Kea**, **Custom**           |
| 💾 Storage             | **Ceph**, Longhorn, etc                      |

</v-click>

</div>
</div>

<!--
Why was Vitistack built?! A trillion-dollar chipmaker buying the dominant enterprise hypervisor company, with a Kubernetes platform included, wanted more money and less features to the users

Key point: you describe WHAT you want (a cluster), the platform figures out HOW.
Everything is a Kubernetes resource -> GitOps, kubectl, RBAC all just work.
CRDs: Vitistack, Machine, MachineClass, MachineProvider, KubernetesProvider,
KubernetesCluster, NetworkNamespace, NetworkConfiguration, IPAllocation, EtcdBackup ...
-->

---

# Kubernetes operators — control loop pattern 🔁

<div class="mt-2">
  <ControlLoop />
</div>

<div class="mt-4 text-center text-lg">

🤖 Operator = **CRD** (the API) + **controller** (the loop)

</div>

<!--
The animation runs by itself (starts when the slide is shown):
1. Observe -> Diff: in sync, nothing to do
2. A worker VM crashes -> drift -> Act creates a new Machine -> self-healed
3. Someone changes replicas 3 -> 5 -> drift -> Act creates 2 Machines -> in sync

Talking points:
- Desired state = spec (what you kubectl apply)
- Actual state = the real world (VMs, IPs, nodes)
- Reconcile until they match, then keep watching
- Level-triggered, idempotent, self-healing
- Progress reported back in status
- Think thermostat: you set 21 degrees, it keeps checking and adjusting.
- Vitistack is a set of operators chained together: one resource's output is the next one's input.
-->

---

# <K8sLogo /> Clusters and Operators 🔁

<img src="/vitistack-operators.svg?v=3" class="h-105 mx-auto mt-2" />

<!--
Source: diagrams/vitistack-operators.excalidraw
- Supervisor cluster runs the Vitistack operators
- Vitistack-operator: core
- Talos-operator: bootstraps Talos Kubernetes on the machines
- Proxmox-operator: creates VMs (Vitistack Machines / K8s nodes) on one or more Proxmox instances
- Kubevirt-operator: creates VMs on one or more KubeVirt clusters (can also be KubeVirt in the supervisor itself)
- aks-operator: provisions clusters in Azure (AKS)
-->

---
layout: section
---

# How to create a cluster 🛠️

<v-clicks>

<div>

🌐 `NetworkNamespace`

</div>

↓

<div>

<K8sLogo /> `KubernetesCluster`

</div>

</v-clicks>

---

# 🌐 NetworkNamespace

<div class="grid grid-cols-[3fr_2fr] gap-10 mt-4">

<div>
Where the cluster(s) lives on the network

```yaml {all|1-2|4-5|7-9|10-12}
apiVersion: vitistack.io/v1alpha2
kind: NetworkNamespace
metadata:
  name: dev-net-001
  namespace: team-cool
spec:
  ipAllocation:
    provider: static-ip-operator
    type: static
  networkProvisioning:
    provider: nam # custom network provider
```

  <!-- <div v-if="$clicks >= 6" class="nn-item mt-4" :class="{ 'nn-active': $clicks === 6 }">
    <div class="font-bold"> <K8sLogo /> Total clusters per networknamespace</div>
    <div class="text-xl opacity-80">40-ish </div>
    <div class="text-sm opacity-80">IP-range x.x.x.1 -> x.x.x.254 and avg 6 nodes per cluster, gives 254 / 6 = 42,33 ish cluster</div>
  </div> -->
</div>

<div class="flex flex-col gap-2">
  <div v-if="$clicks >= 1" class="nn-item" :class="{ 'nn-active': $clicks === 1 }">
    <div class="font-bold">🏷️ apiVersion / kind</div>
    <div class="text-sm opacity-80">the Vitistack API and the resource type</div>
  </div>
  <div v-if="$clicks >= 2" class="nn-item" :class="{ 'nn-active': $clicks === 2 }">
    <div class="font-bold">👥 name / namespace</div>
    <div class="text-sm opacity-80">one per team, clusters reference it by name</div>
  </div>
  <div v-if="$clicks >= 3" class="nn-item" :class="{ 'nn-active': $clicks === 3 }">
    <div class="font-bold">🔢 ipAllocation</div>
    <div class="text-sm opacity-80">who hands out IPs (static-ip-operator, static)</div>
  </div>
  <div v-if="$clicks >= 4" class="nn-item" :class="{ 'nn-active': $clicks === 4 }">
    <div class="font-bold">🔌 networkProvisioning</div>
    <div class="text-sm opacity-80">who provisions the network (nam)</div>
  </div>
</div>

</div>

<style>
.nn-item {
  border-left: 4px solid #475569;
  padding: 0.25rem 0.8rem;
  border-radius: 0.25rem;
  opacity: 0.45;
  transition: all 0.3s;
  animation: nn-in 0.35s ease-out;
}
.nn-active {
  border-color: #60a5fa;
  background: #1e293b;
  opacity: 1;
}
@keyframes nn-in {
  from { opacity: 0; transform: translateX(12px); }
}
</style>

---

# <K8sLogo /> KubernetesCluster — `spec.data`

🪪 Identity and 📍 placement

<div class="relative" :class="{ 'z-20': $clicks === 5 }">

```yaml {all|1-2|3-4|9|10|11}
apiVersion: vitistack.io/v1alpha1
kind: KubernetesCluster
metadata:
  name: supercluster
spec:
  data:
    clusterId: supercluster-1337
    clusterUid: ce660a80-054e-419f-b22c-b6c66a9710bd
    environment: dev
    networkNamespaceName: dev-net-001
    provider: talos
    datacenter: no-south-az1
    region: south
    zone: az1
    project: simple-project
    workorder: simple-workorder
    workspace: simple-workspace
```

</div>

<div v-if="$clicks === 5" class="mc-backdrop" />

<div v-if="$clicks === 5" class="mc-modal mc-kp">
<div class="mc-title"><K8sLogo /> KubernetesProvider <code>talos</code> <span class="mc-hint">← provider: talos</span></div>

```yaml
apiVersion: vitistack.io/v1alpha1
kind: KubernetesProvider
metadata:
  name: talos-provider
spec:
  displayName: Talos Operator Provider
  providerType: talos
  region: south
  version: 1.36.5
```

</div>

<!--
- clusterId / clusterUid: unique identity of the cluster
- datacenter / region / zone: where it runs
- environment: dev
- networkNamespaceName: links to the NetworkNamespace we just created
- project / workorder / workspace: ownership & cost tracking
- provider: talos -> the Kubernetes distribution / Kubernetes provider
-->

---

# <K8sLogo /> KubernetesCluster — `spec.topology`

🧠 Control plane and 💪 workers

<div class="grid grid-cols-2 gap-6">
<div class="relative" :class="{ 'z-20': $clicks === 1 || $clicks === 2 }">

```yaml {all|5|6|7|8-11}{maxHeight:'380px'}
topology:
  version: 1.36.5
  controlplane:
    architecture: amd64
    machineClass: small
    provider: kubevirt
    replicas: 1
    storage:
      - class: standard
        path: /var/lib/vitistack/kubevirt
        size: 20Gi
    version: 1.36.5
```

</div>
<div class="relative" :class="{ 'z-20': $clicks === 6 || $clicks === 7 }">

```yaml {all|3|10|11|12|14-17}{maxHeight:'380px'}
workers:
  nodePools:
    - name: workerpool
      architecture: amd64
      autoscaling:
        enabled: false
        minReplicas: 1
        maxReplicas: 5
        scalingRules: [cpu]
      machineClass: medium
      provider: kubevirt
      replicas: 1
      taint: []
      storage:
        - class: standard
          path: /var/lib/vitistack/kubevirt
          size: 20Gi
      version: 1.36.5
```

</div>
</div>

<div v-if="[1, 2, 6, 7].includes($clicks)" class="mc-backdrop" />

<div v-if="$clicks === 1" class="mc-modal mc-small">
<div class="mc-title">🧩 MachineClass <code>small</code> <span class="mc-hint">← machineClass: small</span></div>

```yaml
apiVersion: vitistack.io/v1alpha1
kind: MachineClass
metadata:
  name: small
spec:
  category: Standard
  cpu:
    cores: 2
    sockets: 1
    threads: 1
  default: false
  description: Small instance with 2 cores and 8Gi memory
  displayName: Small
  enabled: true
  machineProviders: []
  memory:
    quantity: 8Gi
```

</div>

<div v-if="$clicks === 2" class="mc-modal mc-small">
<div class="mc-title">🖥️ MachineProvider <code>kubevirt</code> <span class="mc-hint">← provider</span></div>

```yaml
apiVersion: vitistack.io/v1alpha1
kind: MachineProvider
metadata:
  name: kubevirt-provider
spec:
  authentication:
    credentialsRef:
      namespace: vitistack
      secretName: kubevirt-provider
    type: credentials
  displayName: kubevirt-provider
  providerType: kubevirt
  region: south
```

</div>

<div v-if="$clicks === 7" class="mc-modal mc-medium">
<div class="mc-title">🖥️ MachineProvider <code>kubevirt</code> <span class="mc-hint">provider →</span></div>

```yaml
apiVersion: vitistack.io/v1alpha1
kind: MachineProvider
metadata:
  name: kubevirt-provider
spec:
  authentication:
    credentialsRef:
      namespace: vitistack
      secretName: kubevirt-provider
    type: credentials
  displayName: kubevirt-provider
  providerType: kubevirt
  region: south
```

</div>

<div v-if="$clicks === 6" class="mc-modal mc-medium">
<div class="mc-title">🧩 MachineClass <code>medium</code> <span class="mc-hint">machineClass: medium →</span></div>

```yaml
apiVersion: vitistack.io/v1alpha1
kind: MachineClass
metadata:
  name: medium
spec:
  category: Standard
  cpu:
    cores: 4
    sockets: 1
    threads: 1
  default: true
  description: Medium instance with 4 cores and 16Gi memory
  displayName: Medium
  enabled: true
  machineProviders: []
  memory:
    quantity: 16Gi
```

</div>

<!--
- version: Kubernetes version for the cluster (and per pool)
- machineClass: t-shirt sizes (small/medium) -> MachineClass CRD defines CPU/memory
- provider: kubevirt -> machines are VMs on KubeVirt (could be proxmox/physical)
- replicas: 1 control plane, 3 workers
- storage: disk per node
- nodePools: many pools, each with own size, autoscaling, taints
-->

---

# 🚀 kubectl apply

<v-clicks>

```bash
# 1. k8s namespace
kubectl create namespace team-cool
```

```bash
# 2. Network 🌐
kubectl apply -f manifests/networknamespace.yaml -n team-cool

⧗ wait for the network to be ready
```

```bash
# 3. Then the cluster
kubectl apply -f manifests/kubernetescluster.yaml -n team-cool
```

```bash
# Watch it come alive 👀
watch -n 1 kubectl get networknamespace,kubernetescluster,machines -n team-cool
```

```bash
# Get kubeconfig for new cluster
viti kc login -o=.
```

```bash
# Set KUBECONFIG
export KUBECONFIG=./kubeconfig-supercluster-1337
```

```bash
# Get nodes
kubectl get nodes
```


<!-- <div class="mt-6 text-center text-lg">

📄📄 Two YAML files → 🎉 a running Talos cluster with 1 control plane + 1 workers

</div> -->

</v-clicks>

<!--
Live demo here if possible.
Show the status fields filling in as the operators reconcile.
-->

---
layout: center
---

# 🗺️ How it fits together

<img src="/vitistack-flow.svg?v=4" class="h-100 mx-auto" />

<!--
Source: diagrams/vitistack-flow.excalidraw (open in excalidraw.com, export SVG to public/vitistack-flow.svg)
Walk through:
1. NetworkNamespace -> network provisioning + IP allocation
2. KubernetesCluster -> Kubernetes provider (Talos)
3. Machines -> machine provider (KubeVirt) -> VMs
4. Talos bootstraps control plane + workers -> kubeconfig
-->

---
layout: center
class: text-center
---

# 🐉 Tested at scale

<v-clicks>

<div>
  <img src="/clusters-vms.png" />
</div>

<div>
Regions: 3 (west, central, east, soon north)
</div>

<div>
But also home lab:
<img src="/homelab.jpg" class="h-60 mx-auto mt-2">
</div>

</v-clicks>

---

# 💬 Experiences using Vitistack

<v-clicks>

- 👍 Clusters as YAML → GitOps, review, repeatability
- 👍 Pluggable providers — swap KubeVirt / Proxmox
- 👍 Talos: immutable, API-driven nodes
- 👍 Scales fast
- 🤔 Lessons learned …
  - Naming standards
  - Kea DHCP was not good
  - Need of overview beyond regions
- 🤔 What was hard …
  - Kubernetes knowledge mountain
  - Golang programming
  - Integrations with custom parts

</v-clicks>

<!--
TODO: fill in own experiences
-->

---

# 🔮 The future of Vitistack

<v-clicks>

- 🧩 Template based machine config
- 🔌 More providers (machines and Kubernetes distributions, europe based)
- 📀 More OSes (debian, ubuntu, etc)
- 📈 Autoscaling of node pools
- 💾 Automatic backup & restore (EtcdBackup)
- 📚 Better docs and architecture guides
- 🤝 Community & contributions

</v-clicks>

<!--
TODO: adjust to the actual roadmap
-->

---
layout: center
class: text-center
---

<img src="/viti-logo.svg" class="h-24 mx-auto mb-4" />

# Thanks! 🙏

❓ Questions & 💡 feedback welcome

<div class="mt-10 grid grid-cols-3 gap-8 text-sm">
<div>

🌐 **vitistack.io**

</div>
<div>

📖 **docs.vitistack.io** / **docs.vitistack.io/getting_started**

</div>

<div>

🐙 **github.com/vitistack**

</div>
</div>

<div class="mt-10 opacity-80">
👋 Roger Westerbø · ✉️ roger.westerbo@gmail.com
</div>

<div class="mt-10 opacity-80">
See this presentation on <a href="https://viti.westerbo.no">https://viti.westerbo.no</a>
</div>
