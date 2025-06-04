# Ansible
Ansible is an open-source IT automation tool used to configure systems, deploy applications, and orchestrate tasks across infrastructure. It's agentless, declarative, and uses YAML-based playbooks to describe automation tasks.

🧠 Key Concepts
Concept	Description
Agentless	No software needs to be installed on target machines. Uses SSH or WinRM.
Playbooks	YAML files that define tasks to be executed.
Inventory	List of hosts or groups of hosts where tasks will run.
Modules	Reusable, idempotent units of work (e.g., copy, apt, ios_config).
Idempotent	Running the same playbook multiple times results in the same outcome.
Roles	Way to organize playbooks and tasks into reusable components.

✅ Use Cases
Push network configs to switches and routers (e.g., Cisco, Juniper).
Automate VLAN creation or BGP/OSPF setup.
Backup configurations regularly.
Validate network states (interface status, routing tables).

🔄 Example Scenario: Automating Switch Provisioning
Problem: Engineers manually configure 100+ switches in different DCs.

Solution with Ansible:
Create an inventory with IPs and hostnames of all switches.
Write a playbook using the ios_config module for Cisco devices.
Push configs (hostname, VLANs, SNMP, NTP, etc.) in a repeatable, error-free way.

yaml
Copy
Edit
- name: Configure Cisco Switch
  hosts: cisco_switches
  gather_facts: no
  tasks:
    - name: Set hostname and VLANs
      ios_config:
        lines:
          - hostname Switch01
          - vlan 10
          - name Engineering
      
🚀 Why It’s Valuable for a Head of NRE Provisioning
Scalability: Apply changes across thousands of devices consistently.
Efficiency: Reduce human errors and deployment times.
Auditability: Every change is documented in code.
Standardization: Enforce configuration standards across the network.
Integration: Works well with CI/CD pipelines (e.g., GitLab, Jenkins).

Absolutely! Ansible is a powerful tool for **network automation**, especially useful in roles like **Head of NRE Provisioning**, where efficiency, scalability, and repeatability are key.

### 🔧 **Why Use Ansible for Network Automation?**
* **Agentless**: Uses SSH or APIs—no need to install agents on network devices.
* **Supports Multi-vendor**: Works with Cisco, Juniper, Arista, Fortinet, and more.
* **Idempotent**: Ensures safe, repeatable deployments.
* **Human-readable**: YAML syntax makes it easy for teams to collaborate.

### ✅ **Common Network Automation Tasks with Ansible**
1. Provision new network devices (switches, routers, firewalls)
2. Configure VLANs, interfaces, routing protocols (BGP/OSPF)
3. Backup or restore configurations
4. Verify network state (e.g., interface up/down, BGP neighbors)
5. Roll out security policies (ACLs, SNMP, AAA)

## 🧪 Example: Provisioning a Cisco Switch with Ansible
Let’s walk through a **realistic use case** for provisioning Cisco switches in a data center.

### 📁 **Directory Structure**
```bash
network-automation/
├── inventory/
│   └── hosts.yaml
├── playbooks/
│   └── provision_switch.yml
├── group_vars/
│   └── all.yaml

### 🗂️ `inventory/hosts.yaml`
```yaml
all:
  children:
    switches:
      hosts:
        switch01:
          ansible_host: 192.0.2.10
          ansible_network_os: ios
          ansible_connection: network_cli
          ansible_user: admin
          ansible_password: your_password

### 🗃️ `group_vars/all.yaml`
```yaml
vlans:
  - id: 10
    name: engineering
  - id: 20
    name: hr

### 📜 `playbooks/provision_switch.yml`
```yaml
- name: Provision Cisco Switch
  hosts: switches
  gather_facts: no
  tasks:
    - name: Configure Hostname
      ios_config:
        lines:
          - hostname {{ inventory_hostname }}
    - name: Configure VLANs
      ios_config:
        with_items: "{{ vlans }}"
        lines:
          - vlan {{ item.id }}
          - name {{ item.name }}
    - name: Enable interfaces
      ios_config:
        lines:
          - interface GigabitEthernet1/0/1
          - no shutdown

### ▶️ **Running the Playbook**
```bash
ansible-playbook -i inventory/hosts.yaml playbooks/provision_switch.yml

### 🔍 **Validation Tasks (Optional)**

```yaml
- name: Show VLANs
  ios_command:
    commands: show vlan brief
  register: vlan_output

- debug:
    var: vlan_output.stdout_lines

## 🧠 Best Practices

* Use **role-based structure** for large projects (`ansible-galaxy init`).
* Integrate with **NetBox or IPAM** for dynamic inventory.
* Store secrets securely using **Ansible Vault**.
* Use **GitOps** approach for version control and CI/CD pipelines.

### 🔌 Bonus: Vendor Support via Collections
Install the required network modules (e.g., Cisco):
```bash
ansible-galaxy collection install cisco.ios

--------------------------------------------------------------------------------------------

**Step-by-step procedure** for using **Ansible for network automation** to update configurations on **routers, switches, firewalls, or other network devices**. This approach is modular, scalable, and follows best practices used in large NRE or infrastructure teams.

## 🏗️ **Architecture Overview**
┌────────────────────────┐
│      Source of Truth   │
│ (NetBox / Git / IPAM)  │
└────────────┬───────────┘
             │
             ▼
┌────────────────────────┐
│   CI/CD Pipeline (Optional) │
│ (GitLab CI / Jenkins / AWX)│
└────────────┬───────────┘
             │
             ▼
┌────────────────────────┐
│   Ansible Control Node │
│ (Runs playbooks/YAML)  │
└────────────┬───────────┘
             │SSH/API
             ▼
┌────────────┼───────────┐
│   Network Devices       │
│ (Switches, Routers,     │
│  Firewalls, etc.)       │
└────────────────────────┘

## 🔁 **Step-by-Step Procedure**
### **1. Set Up Your Environment**
✅ **Install Ansible:**
```bash
pip install ansible
```
✅ **Install vendor-specific collections:**
```bash
ansible-galaxy collection install cisco.ios
ansible-galaxy collection install juniper.junos
ansible-galaxy collection install paloaltonetworks.panos
### **2. Prepare the Inventory File**
Create an inventory of your devices.
📄 `inventory/hosts.yaml`
```yaml
all:
  children:
    routers:
      hosts:
        router01:
          ansible_host: 10.10.1.1
          ansible_user: admin
          ansible_password: password
          ansible_network_os: ios
          ansible_connection: network_cli
### **3. Create Configuration Templates**
Use Jinja2 templates to generate configurations dynamically.
📄 `templates/router_config.j2`
```jinja
hostname {{ inventory_hostname }}
interface {{ interface }}
  ip address {{ ip }} {{ subnet }}
  no shutdown
### **4. Create a Playbook**
📄 `playbooks/update_router_config.yml`
```yaml
- name: Push config to Cisco router
  hosts: routers
  gather_facts: no
  vars:
    interface: GigabitEthernet0/1
    ip: 192.168.100.1
    subnet: 255.255.255.0
  tasks:
    - name: Render config
      template:
        src: templates/router_config.j2
        dest: configs/{{ inventory_hostname }}.cfg

    - name: Push configuration to device
      ios_config:
        src: configs/{{ inventory_hostname }}.cfg
### **5. Run the Playbook**
```bash
ansible-playbook -i inventory/hosts.yaml playbooks/update_router_config.yml
### **6. Validate Configuration (Optional)**
📄 Add this to your playbook:
```yaml
- name: Verify interface config
  ios_command:
    commands: show run interface GigabitEthernet0/1
  register: output
- debug:
    var: output.stdout_lines
### **7. Backup Current Configs (Before Changes)**
```yaml
- name: Backup current config
  ios_config:
    backup: yes
  register: backup_result

- debug:
    var: backup_result.backup_path
## 🔐 **Security & Secrets Management**

Use **Ansible Vault** to encrypt sensitive credentials:
```bash
ansible-vault encrypt group_vars/all.yaml
```
---
## 🧩 **Optional Integrations**

| Tool                  | Purpose                                         |
| --------------------- | ----------------------------------------------- |
| **NetBox**            | Source of truth (device IPs, roles, tags)       |
| **GitLab CI/Jenkins** | Automate playbook runs after approval or commit |
| **AWX/Ansible Tower** | GUI-based job control, RBAC                     |


