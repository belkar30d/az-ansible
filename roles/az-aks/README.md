Role Name
=========

A brief description of the role goes here.
Good — this is exactly how a DevOps engineer should approach Ansible (read → extract → design → implement). I’ll break it into 3 parts:

✅ 1. Requirements (from official doc)

From the module doc you shared, the mandatory prerequisites are:

🔹 System Requirements
Python ≥ 2.7

Ansible collection:

ansible-galaxy collection install azure.azcollection

Install Python dependencies:

pip install -r ~/.ansible/collections/ansible_collections/azure/azcollection/requirements.txt

👉 These are required because Azure modules depend on Azure SDK

🔹 Authentication (VERY IMPORTANT)

You must authenticate using one of these:

Option 1: Azure CLI (easiest)
az login
Option 2: Service Principal
export AZURE_SUBSCRIPTION_ID=xxx
export AZURE_CLIENT_ID=xxx
export AZURE_SECRET=xxx
export AZURE_TENANT=xxx
Option 3: Credentials file
~/.azure/credentials

👉 Ansible supports multiple auth sources: CLI, env vars, credential file, MSI

✅ 2. What info you MUST gather before writing playbook

This is where most people fail ❌
Before writing azure_rm_aks, you must collect:

🔷 A. Basic AKS Info
name → AKS cluster name
resource_group
location
dns_prefix
🔷 B. Kubernetes Config
kubernetes_version
enable_rbac (true/false)
🔷 C. Linux Profile (mandatory)
linux_profile:
  admin_username: azureuser
  ssh_key: <public key>

👉 SSH key is required for node access

🔷 D. Identity / Authentication

Choose ONE:

Option 1 (Recommended):
identity:
  type: SystemAssigned
Option 2:
service_principal:
  client_id:
  client_secret:

👉 You cannot use both together

🔷 E. Node Pool (CRITICAL)
agent_pool_profiles:
  - name: nodepool1
    count: 2
    vm_size: Standard_DS2_v2

Required fields:

name
count
vm_size

Optional:

autoscaling (min/max)
availability zones
🔷 F. Network Config
network_profile:
  network_plugin: azure

Options:

azure (CNI)
kubenet
🔷 G. Add-ons (Optional but common)
Monitoring (Log Analytics)
Key Vault
HTTP routing
✅ 3. Pre-Validation / Info Gathering (BEST PRACTICE)

Before creating AKS, always validate environment 👇

🔹 Step 1: Check Azure login
az account show
🔹 Step 2: Verify Resource Group
- name: Get RG info
  azure.azcollection.azure_rm_resourcegroup_info:
    name: myRG
🔹 Step 3: Check existing AKS (important)

Use info module:

- name: Get AKS info
  azure.azcollection.azure_rm_aks_info:
    resource_group: myRG
    name: myAKS
  register: aks_info

👉 This avoids duplicate creation

🔹 Step 4: Validate VNet/Subnet

If using custom networking:

subnet must exist
no IP conflicts
🔹 Step 5: Validate quotas

Check:

VM cores
region availability
✅ 4. Final Process (Production Approach)
🔥 Step-by-step workflow:
Install dependencies
Authenticate to Azure
Gather info:
RG
VNet/Subnet
Existing AKS
Validate inputs
Create AKS using azure_rm_aks
Verify deployment
✅ 5. Minimal Working Playbook
- hosts: localhost
  connection: local
  tasks:

    - name: Create AKS Cluster
      azure.azcollection.azure_rm_aks:
        name: myAKS
        resource_group: myRG
        location: eastus
        dns_prefix: myaksdns
        kubernetes_version: "1.27.7"

        linux_profile:
          admin_username: azureuser
          ssh_key: "{{ ssh_public_key }}"

        identity:
          type: SystemAssigned

        agent_pool_profiles:
          - name: nodepool1
            count: 2
            vm_size: Standard_DS2_v2

        network_profile:
          network_plugin: azure
🔥 Real DevOps Insight (Important for Interview)

Before writing playbook:

❌ Don’t jump into coding
✅ First gather infra dependencies

Think like:
👉 “What AKS needs before creation?”

Answer:

Resource Group
Network
Identity
Node config
Security