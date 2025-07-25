
<img align="right" height="216" src="https://cloud.protogen.engineering/public.php/dav/files/6PPN3gmR75Ccqmc/"  />
<br clear="bottom">
<br clear="bottom">
<br clear="bottom">
<h1 align="left">Paw Dance</h1>



###
<p align="left">a stealth‑grade, post‑quantum SSH VPN</p>

###

<p align="left">Pawdance is a tool that uses OpenSSH you already trust into a fully working Layer‑3 VPN.</p>

###

<h4 align="left">No fixed packet signature</h4>

###

<p align="left">WireGuard and SSTP send a recognisable first‑flight; OpenVPN’s TLS ClientHello can be fingerprinted.<br>SSH randomises its initial IV and padding, so every session’s first packet length is different, defeating simple length‑based fingerprints.</p>

###

<h4 align="left">Stealthy</h4>

###

<p align="left">http://witch.valdikss.org.ru/ test detected as internet modem.</p>

###

<div align="left">
  <img height="400" src="https://cloud.protogen.engineering/public.php/dav/files/CFCC6qL2JR2jfNY"  />
</div>

###


---

## Installation client and server

```bash
# run installer on each side
sudo bash install.sh
```

The installer simply copies `pawdance` into `/usr/local/bin/`
---

## 1 – Prepare the client

```bash
# generate a commented template
pawdance make-config --role client -o pawdance-client.conf

# edit it
vim pawdance-client.conf
```

Example **client** config:

```bash
# pawdance client example config
ROLE="client"

# How to reach the server
CONNECT_MODE="dns"           # dns | ip | auto
REMOTE_HOST="vps.your.domain"
# REMOTE_CONNECT_IP4="203.0.113.42"
# REMOTE_CONNECT_IP6="2001:db8::42"
CONNECT_PREFER="ipv4"         # auto | ipv4 | ipv6

REMOTE_USER="stinky"

# Tunnel interface
TUN_INDEX="1"
TUN_DEV="tun${TUN_INDEX}"

LOCAL_IP4="10.0.1.2/24"
REMOTE_IP4="10.0.1.1"

LOCAL_IP6="2001:db8:1::2/64"
REMOTE_IP6="2001:db8:1::1"

MTU="1500"

# Optional: post‑quantum crypto overrides
SSH_KEX="mlkem768x25519-sha256"
SSH_CIPHERS="chacha20-poly1305@openssh.com"
SSH_MACS="hmac-sha2-512-etm@openssh.com,hmac-sha2-256-etm@openssh.com"

# Push whole‑internet routes through the tunnel?
DEFAULT_ROUTE_IPV4="true"
DEFAULT_ROUTE_IPV6="true"
```

---

## 2 – Prepare the server

```bash
pawdance make-config --role server -o srv-config.conf
vim srv-config.conf
```

Example **server** config:

```bash
ROLE="server"

TUN_INDEX="1"
TUN_DEV="tun${TUN_INDEX}"

LOCAL_IP4="10.0.1.1/24"
LOCAL_IP6="2001:db8:1::1/64"
MTU="1500"

# allow VPN clients to access other networks?
VPN_FORWARD="true"   # adds iptables/ip6tables FORWARD rules

# keep this true (required for routing)
IP_FORWARD="true"    # sets net.ipv4.ip_forward + net.ipv6.conf.all.forwarding
```

---

## 3 – Bring the tunnel up

### On the server

```bash
sudo pawdance up --config srv-config.conf
```

server is now ready. client can connect.

### On the client

```bash
sudo pawdance up --config pawdance-client.conf
```

First run may prompt for:

* *“Are you sure you want to continue connecting (yes/no)?”*  
* SSH password or pass‑phrase (unless key‑based auth already set up)

Once authenticated:

```bash
ip addr show tun1          # should list 10.0.1.2/24
ping 10.0.1.1              # ping the server’s tunnel IP
curl ifconfig.me           # should show the VPS public IP if default routed
```

---

## 4 – Tear down

```bash
# either side:
sudo pawdance down --config <your‑config>.conf
```

This removes:

* per‑family default routes  
* passthrough routes to the SSH endpoint  
* the TUN interface  
* any iptables/ip6tables **FORWARD** rules added by Pawdance  

(Kernel forwarding sysctls remain as you set them.)

---

### Why Pawdance is stealthier than “normal” VPNs

1. **Looks like vanilla SSH** — no OpenVPN/WireGuard/IPsec signatures. 
3. **Randomised first‑packet length** — SSH padding defeats length‑marker DPI.  
4. **Nothing new listening** — only your hardened sshd.  
5. **PQ‑safe handshake** — same post‑quantum KEX most modern OpenSSH clients now use.
---
