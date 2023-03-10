OpenVPN with XOR patch
=========

Ansible role that installs an openvpn server
Install and setup:

* openvpn
* XOR patches
* radius PAM plugin

Requirements
------------

Since  role not supported generating certificates and keys
the users of the role are expected to use some other way of generating certificates/keys (eg using another Ansible role).
For more information, see [Setting up your own Certificate Authority (CA)](https://openvpn.net/community-resources/setting-up-your-own-certificate-authority-ca/)
And more information for [openvpn_xorpatch](https://tunnelblick.net/cOpenvpn_xorpatch.html)

1. Correct /etc/apt/sources.list
2. Correct netplan settings
3. Installed iptables
4. Enabled and started netfilter-persistent.service
5. Add (obfuscate xormask) option to server.conf (password can be generated with "**openssl rand -base64 24**")
6. We have certificate files in [files] role path:

* files:
  * ca.crt
  * crl.pem
  * dh.pem
  * server.crt
  * server.key
  * ta.key

Role Variables
--------------

```yaml
ovpnxor_ovpn_version: 2.5.3
ovpnxor_tunnelblick_version: 3.8.7beta02
vpn_server_ip_address: 192.168.255.1
ovpnxor_ovpn_dns_1: "{{ vpn_server_ip_address }}"
ovpnxor_ovpn_dns_2: 8.8.8.8
ovpnxor_ovpn_listen_port: 6561
# Radius server variables
ovpnxor_radius_server_pwd: you-password
ovpnxor_radius_server_ip1: 10.10.0.1
ovpnxor_radius_server_timeout1: 10
ovpnxor_radius_server_ip2: 10.10.0.2
ovpnxor_radius_server_timeout2: 30
```

Dependencies
------------

[Generating certificates and keys](https://openvpn.net/community-resources/setting-up-your-own-certificate-authority-ca/)

Example Playbook
----------------

```yaml
- name: Configure environment
  hosts: 
    - ovpnxor
  gather_facts: True
  become: True
  force_handlers: True
  roles:
    - { role: ovpnxor }
```

Example user config
------------------

```bash
scramble obfuscate {{ ovpnxor_scramble_obfuscate}}
scramble xormask  {{ ovpnxor_scramble_xormask }}
scramble reverse
client
nobind
dev tun
tun-mtu 1472
mssfix 1300
remote-cert-tls server
tls-client
auth SHA512
auth-nocache
auth-user-pass au.z
cipher AES-256-GCM
remote 10.10.10.10 6561 udp
connect-retry 0 0
connect-retry-max 1
connect-timeout 10
<key>
-----BEGIN PRIVATE KEY-----
...
-----END PRIVATE KEY-----
</key>
<cert>
-----BEGIN CERTIFICATE-----
...
-----END CERTIFICATE-----
</cert>
<ca>
-----BEGIN CERTIFICATE-----
...
-----END CERTIFICATE-----
</ca>
<tls-crypt>
-----BEGIN OpenVPN Static key V1-----
...
-----END OpenVPN Static key V1-----
</tls-crypt>
redirect-gateway def1

```
