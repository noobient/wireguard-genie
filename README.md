# About

WireGuard Genie is a configuration generator for [WireGuard](https://www.wireguard.com/). WireGuard is an awesome piece of software from exceptionally talented people, but their deployment approaches appear to be somewhat lacking. Manually setting up and maintaining such a "server"\* requires lot of manual labor. I made the WireGuard Genie script and the corresponding Ansible playbook to make WireGuard installation and configuration much more streamlined.

\* I put that in quotes, because WireGuard actually doesn't make a distinction between a "server" and "clients", there's only "peers".

## Features

- Automatic, persisent config generation for a set number of clients
- Generated pre-shared keys for added security
- QR code generation for mobile clients
- Peer nick names
- Config versioning in Git
- Config deployment to GitHub and compatible hosting providers
- Split or full tunnelling
- firewalld integration

## Limitations

- Tunnel subnet is always /24
  - Therefore maximum number of clients is 253
- WireGuard server always uses first IP on subnet regardless of last octet in config
- Fresh installs of Fedora Server 35 on Vultr are unable to start firewalld upon first boot:

```
ERROR: Exception DBusException: org.freedesktop.DBus.Error.AccessDenied: Request to own name refused by policy
```

In this case, either `systemctl restart dbus.service firewalld.service`, or reboot the VM once. firewalld will start correctly upon consecutive boots, so this is only needed on the very first boot of the VM, and is probably caused by Vultr's cloud-init procedures.

# Requirements

All you need is a current release of:

- Fedora
- Ubuntu
- CentOS / AlmaLinux / RockyLinux
- RHEL

# Installation

## Automatic

This is the recommended installation method. **Be warned** that the installer will:

- Update all packages on the target server to their latest versions
- Enable automatic updates
- Enable firewalld

This is to ensure that the host handling all your secure communication really is secure and up to the task. Therefore the automatic installation method is best suited for WireGuard "appliances", i.e. hosts whose sole purpose is to handle WireGuard connections, because the points above may disrupt other activities on the server.

### Local

- [Install Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#installing-ansible-on-specific-operating-systems) on the WireGuard server.
- Obtain the WireGuard Genie sources:
```
git clone --recurse-submodules https://github.com/noobient/wireguard-genie.git
```
- Run the installer playbook:
```
ansible-playbook ansible/wireguard.yml
```

### Remote

As the WireGuard Genie installer uses [Ansible](https://www.ansible.com/), it can be done either directly on the WireGuard server or from a remote client, but for the sake of simplicity, this guide will only cover the direct installation part. If you're familiar with Ansible, and have set up your host inventory, private keys, etc. feel free to use the playbooks on remote hosts. The only thing you need to change is specify the target during the playbook run, i.e.

```
ansible-playbook ansible/wireguard.yml -e "target=YOUR.WIREGUARD.SERVER.IP"
```

## Manual

An example installation on Fedora:

```
dnf install wireguard-tools qrencode git
curl https://raw.githubusercontent.com/noobient/wireguard-genie/main/src/wg-gen.sh -o /etc/wireguard/wg-gen.sh
chmod +x /etc/wireguard/wg-gen.sh
```

Then set up `/etc/wireguard/wg-gen.conf` as explained in the [Use](#Use) section. Example:

```
wg_endpoint=YOUR.WIREGUARD.SERVER.IP
wg_ip=10.10.10.1
wg_port=44444
wg_clients=5
wg_dns=1.1.1.1
wg_tunnel='split'
```

Apply the changes with:

```
/etc/wireguard/wg-gen.sh
```

# Use

The default configuration should work OOTB, but you might want to adjust yours. The WireGuard Genie config file is located at `/etc/wireguard/wg-gen.conf`. The options are listed below, with mandatory ones marked with **bold**:

| Key | Description | Default Value |
|---|---|---|
| **`wg_endpoint`** | The FQDN / IP address of your WireGuard server. Clients will connect to this address. | Your server's primary IP address |
| **`wg_ip`** | The "virtual" IP address of the WireGuard Server on the tunnel network. Clients will be assigned adjacent IPs automatically. | 10.10.10.1 |
| **`wg_port`** | The WireGuard server port | 44444 |
| **`wg_clients`** | The number of allowed clients. | 5 |
| **`wg_dns`** | The DNS server clients will use for name resolution. | 1.1.1.1 (Cloudflare) |
| **`wg_tunnel`** | Tunnel type, can be either `full` (all client traffic goes through the server) or `split` (only tunnel traffic goes through). | split |
| `wg_users` | Comma separated list of nick names for your users. Must be the same number as `wg_clients`. | john,jane,jules,juan,jose |
| `wg_repo` | HTTP URL of the GitHub repo where you want to push your generated WireGuard config files. | N/A |
| `wg_credential` | Your GitHub [personal access token](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token) for the repo specified in `wg_repo`. | N/A |

Once you adjusted everything to your needs, you can apply the changes with:

```
/etc/wireguard/wg-gen.sh
```

You can distribute the resulting WireGuard tunnel files to your users from the `/etc/wireguard/clients.d` folder. If you set the `wg_users`, `wg_repo` and `wg_credential` options, the files will be available in your GitHub repo as well.
