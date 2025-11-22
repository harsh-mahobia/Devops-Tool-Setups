
# Adding Some Monitoring Metrics to an Existing Nagios Server

> You can check the link below to create the full Nagios server–client architecture from scratch using AWS:  
**Link:** _https://github.com/harsh-mahobia/Devops-Tool-Setups/blob/main/Nagios-Server-Monitoring.md_

This guide adds the following monitoring metrics:

1. **CPU monitoring**
2. **Memory monitoring**

---

## CPU Monitoring and Disk Monitoring

Below are **clean, organized instructions** with separated commands for:

- **Remote Server**
- **Nagios Server**

This is the final corrected setup including CPU + Memory monitoring.

---

# A. COMMANDS TO RUN ON REMOTE SERVER

This is the server being monitored.

---

## 1. Install Nagios Plugins

```bash
sudo apt update
sudo apt install -y nagios-plugins-basic
````

---

## 2. Create Memory Monitoring Script

```bash
sudo nano /usr/lib/nagios/plugins/check_mem
```

Paste:

```bash
#!/bin/bash
mem_used=$(free | awk '/Mem:/ {print $3}')
mem_total=$(free | awk '/Mem:/ {print $2}')
mem_percent=$(( 100 * mem_used / mem_total ))

if [ $mem_percent -gt 90 ]; then
    echo "CRITICAL - Memory usage at ${mem_percent}%"
    exit 2
elif [ $mem_percent -gt 75 ]; then
    echo "WARNING - Memory usage at ${mem_percent}%"
    exit 1
else
    echo "OK - Memory usage at ${mem_percent}%"
    exit 0
fi
```

Save → **CTRL+O → Enter → CTRL+X**

Make executable:

```bash
sudo chmod +x /usr/lib/nagios/plugins/check_mem
```

---

## 3. Add NRPE Command Definitions

```bash
sudo nano /etc/nagios/nrpe.cfg
```

Add these at the bottom:

```cfg
command[check_cpu]=/usr/lib/nagios/plugins/check_load -w 1.0,1.5,2.0 -c 2.0,3.0,4.0
command[check_memory]=/usr/lib/nagios/plugins/check_mem
```

Save & exit.

---

## 4. Restart NRPE

```bash
sudo systemctl restart nagios-nrpe-server
```

---

## 5. Test Locally (Optional)

CPU:

```bash
/usr/lib/nagios/plugins/check_load -w 1.0,1.5,2.0 -c 2.0,3.0,4.0
```

Memory:

```bash
/usr/lib/nagios/plugins/check_mem
```

---

## 6. Test NRPE from Nagios Server

```bash
/usr/lib/nagios/plugins/check_nrpe -H 172.31.45.132 -c check_cpu
/usr/lib/nagios/plugins/check_nrpe -H 172.31.45.132 -c check_memory
```

---

# B. COMMANDS TO RUN ON NAGIOS SERVER 
This is your Nagios Core dashboard server.

---

## 1. Edit the Monitored Host Config

```bash
sudo nano /usr/local/nagios/etc/servers/remote-server.cfg
```

Add the following service definitions:

### CPU Monitoring

```cfg
define service {
    use                 generic-service
    host_name           remote-server
    service_description CPU Load
    check_command       check_nrpe!check_cpu
}
```

### Memory Monitoring

```cfg
define service {
    use                 generic-service
    host_name           remote-server
    service_description Memory Usage
    check_command       check_nrpe!check_memory
}
```

Save & exit.

---

## 2. Validate Nagios Configuration

```bash
sudo /usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
```

Expected:

```
Total errors: 0
Total warnings: 0
```

---

## 3. Restart Nagios

```bash
sudo systemctl restart nagios
```

---

## 4. Verify in Nagios UI

Open:

```
http://<nagios-public-ip>/nagios
```

Navigate to:

**Services → remote-server**

You should now see:

* ✔ CPU Load — OK
* ✔ Memory Usage — OK

---

# 🎉 Done — CPU + Memory Monitoring Fully Configured!

```


