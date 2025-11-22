
# Ansible problem statement 1 : 

```You are introducing Ansible into your infrastructure for automation. Install Ansible on a control node and create an inventory file listing managed hosts. Run an ad-hoc command to install Apache on all hosts and verify the status using Ansible’s -m service module```

## **GOAL**

1. Launch EC2 instances (one control node + 1–2 managed nodes)
2. Install Ansible on the control node
3. Configure inventory with managed hosts
4. Run an ad-hoc Ansible command to install Apache
5. Verify Apache status using Ansible

---

# **STEP 1 — Launch AWS EC2 instances**

We need:

### **Control Node (Ansible master)**

* Ubuntu 22.04
* Security group: allow SSH (22)

### **Managed Nodes (targets)**

* Ubuntu 22.04
* Security group: allow:

  * SSH (22) **from control node only**
  * Port 80 (HTTP) for Apache test (optional)

### **Generate one key pair**

Use the same key pair for all EC2 instances.

---

# **STEP 2 — SSH into Control Node & Install Ansible**

SSH into the control node:

```bash
ssh -i mykey.pem ubuntu@<CONTROL_NODE_PUBLIC_IP>
```

Install Ansible:

```bash
sudo apt update
sudo apt install ansible -y
```

Check installation:

```bash
ansible --version
```

---

# **STEP 3 — Set up SSH access to managed nodes**

On the control node:

### 1. Copy your `.pem` key

Upload the same key we downloaded from AWS.

Give it correct permissions:

```bash
chmod 400 mykey.pem
```

### 2. SSH into managed nodes (test)

```bash
ssh -i mykey.pem ubuntu@<MANAGED_NODE_PUBLIC_IP>
```

If it works → good.

If it's private IPs only, make sure:

* They are in the same VPC
* Security groups allow SSH from control node’s private IP.

---

# **STEP 4 — Create an Inventory File**

Create a directory:

```bash
mkdir ansible-demo
cd ansible-demo
```

Create **inventory.ini**:

```ini
[webservers]
node1 ansible_host=<MANAGED_NODE1_PUBLIC_IP> ansible_user=ubuntu ansible_ssh_private_key_file=./mykey.pem
node2 ansible_host=<MANAGED_NODE2_PUBLIC_IP> ansible_user=ubuntu ansible_ssh_private_key_file=./mykey.pem
```

---

# **STEP 5 — Test connectivity**

Run a ping:

```bash
ansible -i inventory.ini webservers -m ping
```

Expected output:

```
node1 | SUCCESS => {"ping": "pong"}
node2 | SUCCESS => {"ping": "pong"}
```

If we get SSH errors → means security group issue related to ports.

---

# **STEP 6 — Install Apache using ad-hoc command**

Run this:

```bash
ansible -i inventory.ini webservers -b -m apt -a "name=apache2 state=present update_cache=yes"
```

Explanation:

* `-b` = become sudo
* `-m apt` = use apt module
* `state=present` = install

---

# **STEP 7 — Verify Apache status**

Using the service module:

```bash
ansible -i inventory.ini webservers -b -m service -a "name=apache2 state=started"
```

To check status:

```bash
ansible -i inventory.ini webservers -b -m service -a "name=apache2 state=started enabled=yes"
```

---

| Flag / Argument    | Meaning                       |
| ------------------ | ----------------------------- |
| `-i inventory.ini` | Use this inventory file       |
| `webservers`       | Target host group             |
| `-b`               | Run as sudo (become root)     |
| `-m apt`           | Use apt package module        |
| `-m service`       | Use service management module |
| `-a "..."`         | Pass arguments to the module  |
| `name=apache2`     | Package or service name       |
| `state=present`    | Install the package           |
| `update_cache=yes` | Run `apt update` first        |
| `state=started`    | Start the service             |
| `enabled=yes`      | Enable service at boot        |


# **STEP 8 — Verify manually (optional)**

Open browser:

```
http://<MANAGED_NODE_PUBLIC_IP>
```

We should see the Apache default page.

---

# **RESULT**

We have successfully:

✔ Installed Ansible
✔ Configured an inventory
✔ Managed AWS EC2 nodes
✔ Installed Apache on all servers using ad-hoc commands
✔ Verified through Ansible’s service module

---

