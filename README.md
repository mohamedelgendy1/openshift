# Installing a Three-Node OpenShift Container Platform Compact Cluster Using the Agent-Based Installer

This procedure deploys a **three-node OpenShift Container Platform compact cluster** on bare metal using the Agent-Based Installer.

A compact cluster consists of:

* **3 Control Plane (Master) nodes**
* All master nodes are also **Worker** nodes (schedulable)
* No dedicated worker nodes are required

## Prerequisites

* Access to the OpenShift web console "https://console.redhat.com/openshift/create/datacenter".
* OpenShift pull secret.
* Three bare metal servers that meet the OpenShift hardware requirements.
* `openshift-install` binary downloaded and available in your system `PATH`.
* `nmstate` installed on the workstation where the installer will run.
* `install-config.yaml` and `agent-config.yaml` prepared for a three-node compact cluster deployment.

---

## Step 1: Download the Agent-Based Installer

1. Log in to the OpenShift web console.
2. Navigate to **Datacenter**.
3. Select **Run Agent-based Installer locally**.

Alternatively:

1. Select **Bare Metal (x86_64)** from the cluster type selection page.
2. Select **Local Agent-based** installation.

---

## Step 2: Download Required Files

Download the following:

* Agent-Based Installer
* OpenShift Pull Secret
* OpenShift Command Line Tools (`oc`)

Extract the installer and place the `openshift-install` binary in a directory included in your system `PATH`.

Example:

```bash
sudo mv openshift-install /usr/local/bin/
```

---

## Step 3: Install nmstate

Install the required dependency:

```bash
sudo dnf install nmstate -y
```

Verify installation:

```bash
which nmstatectl
```

---

## Step 4: Create an Installation Directory

Create a directory to store the installation assets:

```bash
mkdir ~/ocp-agent-install
```

---

## Step 5: Create Configuration Files

Create the following files in the installation directory:

### install-config.yaml

Define:

* Cluster name
* Base domain
* Pull secret
* SSH public key
* Cluster networking configuration
* Platform settings

### agent-config.yaml

Define:

* Three hosts
* Rendezvous host
* Network configuration
* Installation disks
* Static networking configuration (if applicable)

---

## Step 6: Generate the Agent ISO

Generate the bootable agent image:

```bash
openshift-install --dir ~/ocp-agent-install agent create image
```

This command creates:

```text
agent.x86_64.iso
```

> **Note:** Red Hat Enterprise Linux CoreOS (RHCOS) supports multipathing on the primary disk and enables it by default in the generated agent image.

---

## Step 7: Boot the Bare Metal Nodes

Boot all three bare metal servers using the generated ISO:

```text
agent.x86_64.iso
```

The rendezvous host will bootstrap the cluster, and the remaining nodes will automatically join during installation.

---

## Step 8: Monitor Bootstrap Progress

To monitor the bootstrap process, run:

```bash
openshift-install --dir ~/ocp-agent-install agent wait-for bootstrap-complete --log-level=info
```

Example output:

```text
INFO Bootstrap configMap status is complete
INFO cluster bootstrap is complete
```

The command completes successfully when the Kubernetes API server has been bootstrapped on the control plane nodes.

---

## Step 9: Monitor Installation Progress

Monitor the installation until completion:

```bash
openshift-install --dir ~/ocp-agent-install agent wait-for install-complete
```

Example output:

```text
INFO Install complete!
INFO To access the cluster as the system:admin user:
INFO export KUBECONFIG=<installation_directory>/auth/kubeconfig
```

---

## Step 10: Access the Cluster

Configure the kubeconfig:

```bash
export KUBECONFIG=~/ocp-agent-install/auth/kubeconfig
```

Verify that all three nodes are ready:

```bash
oc get nodes
```

Expected output:

```text
NAME       STATUS   ROLES                         AGE
master-0   Ready    control-plane,master,worker   XXm
master-1   Ready    control-plane,master,worker   XXm
master-2   Ready    control-plane,master,worker   XXm
```

---

## Deployment Workflow

```text
Download Installer
        ↓
Create install-config.yaml
        ↓
Create agent-config.yaml
        ↓
Generate Agent ISO
        ↓
Boot 3 Bare Metal Nodes
        ↓
Bootstrap Rendezvous Host
        ↓
Cluster Formation
        ↓
Wait for Bootstrap Complete
        ↓
Wait for Install Complete
        ↓
Verify Cluster Health
```
