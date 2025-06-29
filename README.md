# CRC Custom Network Configuration

This guide sets up a **custom `crc` network** for use with CodeReady Containers (CRC) using `libvirt`.

---

## 🔧 Network Configuration File

Create a new libvirt network configuration file:

```bash
cat <<EOF > crc-net.xml
<network>
  <name>crc</name>
  <forward mode='nat'/>
  <bridge name='crc' stp='on' delay='0'/>
  <ip address='192.168.130.1' netmask='255.255.255.0'>
    <dhcp>
      <range start='192.168.130.2' end='192.168.130.254'/>
    </dhcp>
  </ip>
</network>
EOF
```

---

## 📌 Implementation Steps

### 1. Define and Start the Network

```bash
sudo virsh net-define crc-net.xml
sudo virsh net-autostart crc
sudo virsh net-start crc
```

### 2. Verify the Network

```bash
sudo virsh net-info crc
```

Expected output:

```
Name:           crc
UUID:           <some-uuid>
Active:         yes
Persistent:     yes
Autostart:      yes
Bridge:         crc
```

---

## ⚙️ Configure CRC

Set CRC to use system networking and start the cluster:

```bash
crc config set network-mode system
crc setup
crc start -p ./pullsecret.txt
```

---

## 💡 Why This Configuration?

- Uses `192.168.130.0/24` to avoid conflicts with:
  - Your main network: `172.18.99.0/24`
  - Default libvirt network: `192.168.122.0/24`
- Provides NAT isolation while allowing internet access
- Uses standard CRC IP range for compatibility

---

## ✅ Expected Result

After `crc start`:

```bash
crc ip
```

Should return an IP from the `192.168.130.0/24` range, e.g.:

```bash
192.168.130.11
```

---

## 🛠️ Troubleshooting

### ❌ If `crc ip` returns `127.0.0.1`:

1. Check network status:

```bash
sudo virsh net-info crc
```

If it shows:

```
Active:         no
Autostart:      no
```

Then **start and enable autostart**:

```bash
sudo virsh net-start crc
sudo virsh net-autostart crc
```

2. Verify network XML:

```bash
sudo virsh net-dumpxml crc | grep -A10 '<ip address'
```

Ensure it includes the DHCP range.

3. Check if VM uses the correct network:

```bash
sudo virsh dumpxml crc | grep -A5 '<interface'
```

Should include:

```xml
<interface type='network'>
  <source network='crc'/>
</interface>
```

---

## 🧩 Add to `/etc/hosts`

If `crc ip` is working, map CRC domain names locally:

```bash
echo "$(crc ip) api.crc.testing console-openshift-console.apps-crc.testing oauth-openshift.apps-crc.testing" | sudo tee -a /etc/hosts
```

---

## 📁 Example Output

```bash
[v.tavakoli@ILBAPPCM2 ~]$ sudo virsh net-info crc
Name:           crc
UUID:           e27b7626-eb88-405a-a433-8864ba0ab62e
Active:         no
Persistent:     yes
Autostart:      no
Bridge:         crc
```

Use the `net-start` and `net-autostart` commands as shown above to activate it.

---

> 📝 Maintained by [Your Name] - Feel free to contribute or raise issues.
