# Ansible Scenario 2 : 

_This would be easy if you had already performed [Ansible Scenario 1](https://github.com/harsh-mahobia/Devops-Tool-Setups/blob/main/ansible-scenario-1.md)_

> Demonstrate how Ansible simplifies system administration by automating software installation, service management, and conflict resolution across remote servers. Set up an Ansible Controller on an AWS EC2 instance and manage another EC2 instance as the target node through SSH. Create an idempotent YAML playbook that completely removes Apache (if installed and blocking port 80), cleans up leftover processes, installs nginx, and ensures the nginx service is running and enabled at boot.

> In short, We are going to delete Apache Server from all target nodes(if present) and then Install Nginx.

The playbook must remain **idempotent—meaning** running it multiple times should result in no changes when the system is already configured correctly.


---
This is the exact, real-world workflow you performed on AWS to demonstrate how Ansible simplifies system administration.
---

# **1. Launch AWS EC2 Instances**

You create **two Ubuntu 22.04 EC2 instances**:

### **(A) Ansible Controller**

* Will run Ansible
* Security Group: Allow SSH (port 22) from your IP

### **(B) Managed Node (Target Node)**

* Will receive configuration from Ansible
* Security Group: Allow SSH from the controller’s private IP range
  Example: `172.31.0.0/16`

---

# **2. Install Ansible on Controller**

SSH into controller:
```
nano key.pem
```
> copy and paste the content from downloaded .pem file

```bash
ssh -i key.pem ubuntu@<controller-public-ip>
```


Install Ansible:

```bash
sudo apt update
sudo apt install ansible -y
```

Check version:

```bash
ansible --version
```

---

# **3. Create Inventory File**

Inside controller:

```bash
mkdir ansible-demo
cd ansible-demo
nano inventory.ini
```

Add:

```ini
[webservers]
node1 ansible_host=<managed-node-public-ip> ansible_user=ubuntu ansible_ssh_private_key_file=/home/ubuntu/key.pem
```

Save → exit.

**At the bottom we've table to explain you about each tag used here.**

---

# **4. Verify Connection**

```bash
ansible -i inventory.ini webservers -m ping
```

Expected:

```
node1 | SUCCESS => { "ping": "pong" }
```

---

# **5. Create the Final Playbook**

Create file:

```bash
nano install_nginx.yml
```

Paste:

```yaml
---
- name: Remove Apache and install/start nginx
  hosts: webservers
  become: yes

  tasks:
    - name: Ensure apt cache is updated
      apt:
        update_cache: yes
        cache_valid_time: 3600

    - name: Stop apache service if running
      service:
        name: apache2
        state: stopped
      ignore_errors: yes

    - name: Disable apache service
      service:
        name: apache2
        enabled: no
      ignore_errors: yes

    - name: Kill any leftover apache2 processes (port 80 blockers)
      shell: |
        pkill -f apache2 || true
      failed_when: false
      changed_when: false

    - name: Remove apache2 completely (purge)
      apt:
        name: apache2
        state: absent
        purge: yes
      ignore_errors: yes

    - name: Install nginx
      apt:
        name: nginx
        state: present

    - name: Ensure nginx is running and enabled
      service:
        name: nginx
        state: started
        enabled: yes
```

Save → exit.

> **At the bottom we've table to explain you about each tag used here.**

---

# **6. Execute the Playbook**

Run:

```bash
ansible-playbook -i inventory.ini install_nginx.yml
```

Expected result:

```
changed=3 failed=0
```

This means:

* Apache was stopped/removed
* Nginx installed
* Nginx started

---

# **7. Verify Idempotence**

Run again:

```bash
ansible-playbook -i inventory.ini install_nginx.yml
```

Expected:

```
changed=0 failed=0
```

System is already correct → **idempotency achieved**.

---

# **8. Verify nginx is running (Ansible-only)**

```bash
ansible -i inventory.ini webservers -m shell -a "sudo ss -tulpn | grep :80"
```

Expected output:

```
nginx  LISTEN ...
```

Now nginx is serving the default page on port 80.

---

# 🎉 **Final Result**

Ansible:

* automatically removed Apache
* resolved port conflicts
* installed nginx
* ensured nginx is running and enabled
* maintained full idempotency

All without manually logging into the server.

---

# **Table Explaining All Tags / Modules Used**

| Element       | Meaning                        | Why Used                                             |
| ------------- | ------------------------------ | ---------------------------------------------------- |
| `---`         | YAML document start            | Standard syntax                                      |
| `name:`       | Human-readable task/play name  | Improves clarity in output                           |
| `hosts:`      | Specifies target machine/group | Tells Ansible where to run tasks                     |
| `become: yes` | Runs tasks with sudo           | Required for installing packages & starting services |
| `tasks:`      | List of actions to execute     | Core of the playbook                                 |

### **apt Module**

| Key                      | Meaning                                  | Why Used                |
| ------------------------ | ---------------------------------------- | ----------------------- |
| `apt:`                   | Manages packages on Debian-based systems | Install/remove packages |
| `update_cache: yes`      | Runs `apt update`                        | Ensures fresh metadata  |
| `cache_valid_time: 3600` | Only updates if older than 1 hour        | Keeps it idempotent     |
| `name: <package>`        | Package name                             | e.g., nginx, apache2    |
| `state: present`         | Ensures package exists                   | Install nginx           |
| `state: absent`          | Ensures package removed                  | Remove Apache           |
| `purge: yes`             | Removes configs too                      | Complete cleanup        |

### **service Module**

| Key              | Meaning                   | Why Used            |
| ---------------- | ------------------------- | ------------------- |
| `service:`       | Controls systemd services | Start/stop services |
| `name:`          | Service name              | apache2, nginx      |
| `state: stopped` | Stop a running service    | Stop Apache         |
| `state: started` | Start service             | Start nginx         |
| `enabled: no`    | Disable service on boot   | Disable Apache      |
| `enabled: yes`   | Enable service on boot    | Auto-start nginx    |

### **shell Module**

| Key                   | Meaning                                      | Why Used                  |                                 |              |
| --------------------- | -------------------------------------------- | ------------------------- | ------------------------------- | ------------ |
| `shell:`              | Run shell commands                           | Needed for pkill          |                                 |              |
| `pkill -f apache2     |                                              | true`                     | Kill any running Apache process | Free port 80 |
| `failed_when: false`  | Prevents Ansible from marking task as failed | pkill may return non-zero |                                 |              |
| `changed_when: false` | Ensures task never shows as "changed"        | Idempotency               |                                 |              |

### **ignore_errors: yes**

| Meaning                      | Why Used                                       |
| ---------------------------- | ---------------------------------------------- |
| Continue even if task errors | Apache may not exist; avoids breaking playbook |

---

# Thankyou
> Have a Great Learning
