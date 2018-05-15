Role : ansible-role-vault-install
=================================

Installs Hashicorp's vault by
* Downloading & unzipping vault from Hashicorp's releases site into /usr/bin
* Creating a vault group & user
* Creating the vault directory structure
* Installing latest version of pyOpenSSL using pip
* Generating Self-Signed SSL/TLS certificate
* Creating vault systemd service
* Starting & enabling vault systemd service
* Setting VAULT_ADDR & VAULT_CACERT in system-wide profile

Currently tested on these Operating Systems
* Oracle Linux/RHEL/CentOS 7
* Debian/Stretch64

Requirements
------------

Ansible 2.5 or higher
pyOpenSSL 0.17 or higher (see Known Issues section below)
Backend consul cluster (role: ansible-role-consul-install)

Role Variables
--------------

defaults/main.yml
```
vault_src: "https://releases.hashicorp.com/vault/0.10.1/vault_0.10.1_linux_amd64.zip"	# Version of vault to download from Hashicorp's website
vault_chksum: "sha256:031e521b4603487126fd353a9557dd22a02304a8a11f843e9914be59a8009c8a"	# Vault download file checksum
vault_bin_path: "/usr/bin"								# Path to install vault binary
vault_path: "/var/lib/vault"								# The base vault directory
vault_data: "{{ vault_path }}/data"							# Vault's data directory (using 'file' backend storage)
vault_conf: "/etc/vault/config.hcl"							# Vault configuration file
vault_certs: "/etc/vault/certs"								# Vault certificates directory
vault_cn: "{{ ansible_fqdn }}"								# CSR Common Name
vault_cc: "UK"										# CSR Country Code
vault_on: "Vault"									# CSR Organization Name
vault_privkey: "{{ vault_certs }}/{{ vault_cn | regex_replace('^www\\.', '') }}.pem"	# OpenSSL Private Key filename
vault_csr: "{{ vault_certs }}/{{ vault_cn }}.csr"					# OpenSSL Certificate Signing Request filename
vault_certfile: "{{ vault_certs }}/{{ vault_cn | regex_replace('^www\\.', '') }}.crt"	# OpenSSL Certificate filename
vault_base_ip: "10.1.42."								# Vault subnet
vault_addr: "{{ ansible_fqdn }}"							# Vault listener address
vault_port: "8200"									# Vault listener port
consul_port: "8500"									# Consul listener port
vault_user: "vault"									# User to run the vault systemd service
vault_group: "vault"									# Group for vault user
vault_service: "vault"									# Name of the vault systemd service
vault_profile: "/etc/profile.d/vault.sh"						# System-wide profile for setting Vault listening address
```

Dependencies
------------

Requires elevated root privileges

Example Playbook
----------------

```
---

- name: Install Hashicorp Vault
  hosts: vault
  become: yes

  roles:
    - ansible-role-vault-install
```

Known Issues
------------

If you get the message "You need to have PyOpenSSL>=0.15 to generate CSRs", then it is most likely an issue with the OpenSSL package that python has imported. When pyOpenSSL is installed/upgraded via the PIP Ansible module in this playbok, it will install the python package under /usr/lib/pythonx.x/site-packages, however it is possible that another OpenSSL python package could be installed under /usr/lib64/pythonx.x/site-packages that is being loaded in preference to the higher-level package.
In order to prevent this happening, temporarily move the directory "/usr/lib64/pythonx.x/site-packages/OpenSSL" out of the way while running this playbook.

License
-------

BSD

Author Information
------------------

Adam Goldsmith
