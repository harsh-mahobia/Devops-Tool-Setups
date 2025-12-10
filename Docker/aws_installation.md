# **Install Docker using the official script (stable)**

This avoids dependency issues on Ubuntu

```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
```

---

# **Enable & start Docker**

```bash
sudo systemctl enable docker
sudo systemctl start docker
sudo systemctl status docker
```

You should now see **active (running)**.

---

# **Test**

```bash
sudo docker run hello-world
```

---

#  **Check output of:**

```bash
sudo systemctl status docker --no-pager -l
```
