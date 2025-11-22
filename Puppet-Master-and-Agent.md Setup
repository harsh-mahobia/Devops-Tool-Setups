
# **Puppet Master–Agent Deployment Guide (Puppet 8) — AWS Ubuntu 22.04**

This guide sets up:

* **Puppet Master** (puppetserver)
* **Puppet Agent** (client node)
* Automated **Nginx installation + index.html deployment**
* Full troubleshooting flow
* All verification commands

---

# 📌 **1. AWS Setup**

### Create **2 EC2 instances**

* OS: **Ubuntu 22.04 LTS**
* Instance type: **t2.micro**

### Security Group rules (both machines)

| Port         | Purpose        |
| ------------ | -------------- |
| **22/tcp**   | SSH            |
| **8140/tcp** | Puppet Master  |
| **80/tcp**   | Nginx on Agent |

Also allow **internal communication** between the machines.

---

# 📌 **2. Configure Hostnames**

### On Puppet Master

```bash
sudo hostnamectl set-hostname puppetmaster
```

### On Puppet Agent

```bash
sudo hostnamectl set-hostname puppetagent
```

---

# 📌 **3. Install Puppet Master (Puppet 8)**

**SSH into puppetmaster**:

```bash
sudo apt update
sudo apt install wget -y

wget https://apt.puppet.com/puppet8-release-focal.deb
sudo dpkg -i puppet8-release-focal.deb
sudo apt update

sudo apt install puppetserver -y
```

### Adjust memory

```bash
sudo sed -i 's/Xms2g/Xms512m/' /etc/default/puppetserver
sudo sed -i 's/Xmx2g/Xmx512m/' /etc/default/puppetserver
```

### Start Puppet Server

```bash
sudo systemctl enable --now puppetserver
```

---

# 📌 **4. Install Puppet Agent (_on the Master too_)**

The master requires the agent tools for running CLI commands.

```bash
wget https://apt.puppet.com/puppet8-release-focal.deb
sudo dpkg -i puppet8-release-focal.deb
sudo apt update

sudo apt install puppet-agent -y
```

### Add `puppet` to PATH

```bash
echo 'export PATH=/opt/puppetlabs/puppet/bin:$PATH' | sudo tee -a ~/.bashrc
source ~/.bashrc
```

### Verify

```bash
which puppet
puppet --version
```

Should show:

```
/opt/puppetlabs/puppet/bin/puppet
```

---

# 📌 **5. Create Puppet Manifest (site.pp)**

On **puppetmaster**:

```bash
sudo nano /etc/puppetlabs/code/environments/production/manifests/site.pp
```

Paste:

```puppet
node 'puppetagent' {

  package { 'nginx':
    ensure => installed,
  }

  service { 'nginx':
    ensure => running,
    enable => true,
    require => Package['nginx'],
  }

  file { '/var/www/html/index.html':
    ensure  => file,
    content => "<h1>Puppet 8 Automated Nginx Deployment Successful</h1>",
    owner   => 'www-data',
    group   => 'www-data',
    mode    => '0644',
    require => Package['nginx'],
  }

}
```

---

# 📌 **6. Install Puppet Agent (on Agent Node)**

SSH into **puppetagent**:

```bash
sudo apt remove puppet-agent -y
sudo rm /etc/apt/sources.list.d/puppet*.list
sudo apt update

wget https://apt.puppet.com/puppet8-release-focal.deb
sudo dpkg -i puppet8-release-focal.deb
sudo apt update

sudo apt install puppet-agent -y
```

Add the PATH:

```bash
echo 'export PATH=/opt/puppetlabs/puppet/bin:$PATH' | sudo tee -a ~/.bashrc
source ~/.bashrc
```

Verify:

```bash
which puppet
puppet --version
```

---

# 📌 **7. Configure Agent to Communicate with Master**

Find master private IP:

```bash
hostname -I
```

### Edit puppet.conf on Agent:

```bash
sudo nano /etc/puppetlabs/puppet/puppet.conf
```

Add:

```
[main]
server = puppetmaster.ap-south-1.compute.internal
certname = puppetagent
```

### Add master mapping to /etc/hosts:

```bash
sudo nano /etc/hosts
```

Add:

```
172.31.47.27  puppetmaster.ap-south-1.compute.internal puppet
```

(Replace with **your master private IP**)

---

# 📌 **8. First Puppet Run on Agent (will show errors, expected)**

```bash
sudo puppet agent -t
```

Expected output:

```
Certificate not yet signed
```

---

# 📌 **9. Sign the Agent Certificate (on Master)**

On **puppetmaster**:

### List pending certs

```bash
sudo /opt/puppetlabs/server/bin/puppetserver ca list --all
```

### Sign cert

```bash
sudo /opt/puppetlabs/server/bin/puppetserver ca sign --certname puppetagent
```

Or sign all:

```bash
sudo /opt/puppetlabs/server/bin/puppetserver ca sign --all
```

---

# 📌 **10. Final Puppet Run on Agent**

```bash
sudo puppet agent -t
```

Expected:

✔ nginx installed
✔ index.html created
✔ service running
✔ catalog applied

---

# 📌 **11. Verify Nginx**

### Check status:

```bash
sudo systemctl status nginx
```

### Test page:

```bash
curl http://localhost
```

You should see your custom HTML.

### External access:

Open in browser:

```
http://<agent-public-ip>
```

---

# 🛠 **Troubleshooting Cheat Sheet**

### ❌ puppet: command not found (agent or master)

Fix PATH:

```bash
echo 'export PATH=/opt/puppetlabs/puppet/bin:$PATH' | sudo tee -a ~/.bashrc
source ~/.bashrc
```

If still not found:

```bash
sudo ln -s /opt/puppetlabs/puppet/bin/puppet /usr/bin/puppet
```

---

### ❌ Unable to connect to puppet:8140 (hostname error)

Add to agent’s `/etc/hosts`:

```
<MASTER_PRIVATE_IP> puppetmaster.ap-south-1.compute.internal puppet
```

---

### ❌ Hostname mismatch / SSL error

Clear SSL on agent:

```bash
sudo rm -rf /etc/puppetlabs/puppet/ssl/*
```

Clear old cert on master:

```bash
sudo /opt/puppetlabs/server/bin/puppetserver ca revoke --certname puppetagent
sudo /opt/puppetlabs/server/bin/puppetserver ca clean --certname puppetagent
```

Restart puppetserver:

```bash
sudo systemctl restart puppetserver
```

---

### ❌ Puppet agent says certificate not signed

Run:

```bash
sudo /opt/puppetlabs/server/bin/puppetserver ca sign --all
```

---

### ❌ Catalog compilation errors

Check syntax:

```bash
sudo puppet parser validate /etc/puppetlabs/code/environments/production/manifests/site.pp
```

---

### ❌ Master cannot resolve its hostname

Fix master `/etc/hosts`:

```bash
127.0.0.1   puppet puppetmaster puppetmaster.ap-south-1.compute.internal localhost
<MASTER_PRIVATE_IP> puppetmaster.ap-south-1.compute.internal puppetmaster
```

Restart:

```bash
sudo systemctl restart puppetserver
```

---

# **DONE**

This guide now gives you everything you need to:

* Recreate Puppet Master + Agent anytime
* Avoid all previous issues
* Verify every component
* Troubleshoot every failure mode
