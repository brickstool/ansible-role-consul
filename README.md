# Ansible Role: Consul

This role installs and configures a Consul cluster. It aims to have sensible
defaults and to only require a small amount of modification out of the box.

Tested on a 28-node Consul cluster (3 servers) that includes CentOS 7+8, Ubuntu
18.04, and Proxmox systems.

## Requirements

Using this role only requires `netaddr` on the host that is running Ansible. To
install `netaddr`:

```
pip install netaddr
```

Target host compatibility is limited to Linux systems from the RedHat or Debian
families that use `systemd`. Which is essentially every modern Linux system.

## Dependencies

If you do not want to use consul-template in this set-up at all, then there
are no dependencies!

The consul-template (`ctpl`) tasks in `tasks/ctpl.yml` will require the
installation and configuration of consul-template which is outside the scope
for this role. The `systemd` unit file for the consul-template service should
look something like this:

```ini
[Unit]
Description=consul-template for %I
Requires=network-online.target consul.service
After=network-online.target consul.service vault.service nomad.service

[Service]
ExecStart=/usr/local/bin/consul-template -config=/etc/consul-template.d/%I.hcl
ExecReload=/bin/kill -HUP $MAINPID
ExecStop=/bin/kill -INT $MAINPID
PIDFile=/run/consul-template-%I.pid
Restart=always
KillMode=process
KillSignal=SIGINT
RestartSec=60s
LimitNOFILE=4096
TimeoutSec=5

[Install]
WantedBy=multi-user.target
```

Notice the use of `%I` in the example. That allows for dynamically specifying
a configuration file located in `/etc/consul-template.d` when starting the
service. Using the example configuration file in this role
(`templates/ctpl.consul.hcl.j2`) along with the `systemd` unit file block
above, you would execute the following command to enable and start the service:

```sh
$ sudo systemctl enable --now consul-template@consul.service
```

Using consul-template will also require a working HashiCorp Vault cluster with
the PKI secrets engine enabled. This is absolutely outside the scope of this
role, sorry.

## Example Playbook

The following examples are the minimum configuration you would need to
successfully run this role. Generate an actual encryption key for
`consul_encrypt_string` using `consul keygen`, replacing "encryptme123" with
the generated key in the examples below.

*For a Consul server node*:

    - hosts: consul-server
      become: yes
      roles:
        - role: brickstool.consul
          vars:
            consul_server: true
            consul_encrypt_string: 'encryptme123='
            consul_retry_join:
              - 10.0.0.1

*For a Consul client node*:

    - hosts: consul-client
      become: yes
      roles:
        - role: brickstool.consul
          vars:
            consul_encrypt_string: 'encryptme123='
            consul_retry_join:
              - 10.0.0.1

## License

MIT / BSD

## Author Information

Created by Samuel Noordhuis in 2020. Inspired heavily by the Ansible roles and
writings of [Jeff Geerling](https://github.com/geerlingguy).

If you see any errors or think this role could be improved in some way, you
are welcome to open an issue/feature request or create a pull request :)

## TO-DO

1. Add molecule tests
2. Move consul-template tasks into separate role
3. Add additional configuration options for Consul
