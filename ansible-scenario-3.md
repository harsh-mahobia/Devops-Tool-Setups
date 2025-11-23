
# Ansible Scenario 2 : Deploying NGINX With Custom Web Page Using Handlers

*This would be easy if you had already performed [Ansible Scenario 2](https://github.com/harsh-mahobia/Devops-Tool-Setups/blob/main/ansible-scenario-2.md)*

> Demonstrate how Ansible simplifies system administration by automating the deployment of a complete NGINX-based web server on an AWS EC2 instance. Create a YAML playbook that installs NGINX, deploys a custom `index.html` file from the `ansible-demo` directory, and ensures the service runs on startup. Use handlers so that NGINX automatically restarts whenever configuration or web content files change, maintaining idempotency—meaning running the playbook multiple times will not alter the system unless changes are detected.

> In short, we are installing NGINX, deploying our own index.html, enabling the service on boot, and using handlers to restart NGINX automatically whenever files change.

The playbook remains **fully idempotent** — rerunning it results in **no changes** when the system is already configured correctly.

---

# **1. Launch AWS EC2 Instances**

You create **two Ubuntu 22.04 EC2 instances**:

### **(A) Ansible Controller**

* Will run Ansible
* Security Group: Allow SSH (22) from your IP

### **(B) Managed Node (Target Node)**

* Will receive configuration from Ansible
* Security Group: Allow SSH from controller’s private IP range
  Example: `172.31.0.0/16`
* HTTP (port 80) should be allowed publicly

---

# **2. Install Ansible on the Controller**

SSH into controller:

```bash
nano key.pem
```

> Paste content of your downloaded PEM file

```bash
ssh -i key.pem ubuntu@<controller-public-ip>
```

Install Ansible:

```bash
sudo apt update
sudo apt install ansible -y
```

Verify:

```bash
ansible --version
```

---

# **3. Create the Inventory File**

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

*A table explaining each tag is provided at the bottom.*

---

# **4. Verify SSH Connection**

```bash
ansible -i inventory.ini webservers -m ping
```

Expected:

```
node1 | SUCCESS => { "ping": "pong" }
```

---

# **5. Create Custom index.html**

Inside your `ansible-demo` folder:

```bash
nano index.html
```

Example content:

```html
<h1>Welcome to My AWS NGINX Server</h1>
<p>Deployed using Ansible 🚀</p>
```

---

# **6. Create the Final Playbook With Handlers**

Create:

```bash
nano deploy_nginx.yml
```

Paste:

```yaml
---
- name: Deploy NGINX and custom web page
  hosts: webservers
  become: yes

  vars:
    nginx_pkg: nginx
    nginx_service: nginx
    web_root: /var/www/html

  tasks:

    - name: Install NGINX
      apt:
        name: "{{ nginx_pkg }}"
        state: present
        update_cache: yes
      notify: Restart NGINX

    - name: Ensure web root exists
      file:
        path: "{{ web_root }}"
        state: directory
        owner: www-data
        group: www-data
        mode: "0755"

    - name: Deploy custom index.html
      copy:
        src: ./index.html
        dest: "{{ web_root }}/index.html"
        owner: www-data
        group: www-data
        mode: "0644"
      notify: Restart NGINX

    - name: Ensure NGINX is running and enabled
      service:
        name: "{{ nginx_service }}"
        state: started
        enabled: yes

  handlers:
    - name: Restart NGINX
      service:
        name: "{{ nginx_service }}"
        state: restarted
```

Save → exit.

*A detailed table explaining every tag is at the bottom.*

---

# **7. Execute the Playbook**

Run:

```bash
ansible-playbook -i inventory.ini deploy_nginx.yml
```

Expected:

```
changed=X failed=0
```

This means:

* NGINX installed
* Custom web page deployed
* NGINX restarted
* Service enabled

---

# **8. Verify Idempotency**

Run again:

```bash
ansible-playbook -i inventory.ini deploy_nginx.yml
```

Expected:

```
changed=0 failed=0
```

Perfect idempotency ✔

---

# **9. Verify NGINX is Running on Port 80**

```bash
ansible -i inventory.ini webservers -m shell -a "sudo ss -tulpn | grep :80"
```

Expected:

```
nginx   LISTEN ...
```

Then check in browser:

```
http://<managed-node-public-ip>
```

Your custom HTML page should display.

---

# 🎉 **Final Result**

You successfully used Ansible to:

* Install NGINX
* Deploy a custom website
* Restart NGINX automatically via handlers
* Ensure service enabled on boot
* Maintain complete **idempotency**
* Manage everything remotely via Ansible on AWS

---

# **Table Explaining All Tags / Modules Used**

### **Play & YAML Structure**

| Element       | Meaning                | Why Used                                |
| ------------- | ---------------------- | --------------------------------------- |
| `---`         | YAML document start    | Required syntax                         |
| `name:`       | Human-readable name    | Better logging                          |
| `hosts:`      | Target machine/group   | Specifies where playbook runs           |
| `become: yes` | Use sudo               | Required for package/service management |
| `vars:`       | Variables used in play | Cleaner configuration                   |

---

### **Modules Used**

#### **apt Module**

| Key                 | Meaning          | Why Used              |
| ------------------- | ---------------- | --------------------- |
| `apt:`              | Manage packages  | Install NGINX         |
| `name:`             | Package name     | `"nginx"`             |
| `state: present`    | Ensure installed | Correct package state |
| `update_cache: yes` | Run apt update   | Get latest metadata   |

---

#### **copy Module**

| Key      | Meaning                     | Why Used                   |
| -------- | --------------------------- | -------------------------- |
| `src:`   | Source file from controller | Your `index.html`          |
| `dest:`  | Destination on target node  | `/var/www/html/index.html` |
| `owner:` | File owner                  | Required by NGINX          |
| `group:` | Group                       | Ensures proper permissions |
| `mode:`  | File permission             | Secure & appropriate       |

---

#### **file Module**

| Key                | Meaning                 | Why Used                         |
| ------------------ | ----------------------- | -------------------------------- |
| `state: directory` | Ensure directory exists | Make sure `/var/www/html` exists |

---

#### **service Module**

| Key              | Meaning                  | Why Used           |
| ---------------- | ------------------------ | ------------------ |
| `service:`       | Control systemd services | Start/enable NGINX |
| `state: started` | Ensure running           | Starts NGINX       |
| `enabled: yes`   | Auto-start on boot       | Persistence        |

---

### **Handlers**

| Name            | Meaning                            | Why Used                        |
| --------------- | ---------------------------------- | ------------------------------- |
| `Restart NGINX` | Restart service only when notified | Restarts only when file changes |
| `notify:`       | Triggers handler when task changes | Makes playbook efficient        |

---

# Thank you

> Keep Exploring, Keep Automating 🚀

---

