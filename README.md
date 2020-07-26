<p align="center">
  <a href="https://github.com/brickstool/ansible-role-consul/actions">
    <img alt="GitHub Actions" src="https://github.com/brickstool/ansible-role-consul/workflows/build/badge.svg?branch=master">
  </a>
  <a href="https://github.com/semantic-release/semantic-release">
    <img alt="semantic-release" src="https://img.shields.io/badge/%20%20%F0%9F%93%A6%F0%9F%9A%80-semantic--release-e10079.svg">
  </a>
</p>

<h1 align="center" style="border-bottom: none;">Ansible Role: Consul</h1>

This role installs and configures a Consul cluster.
It aims to have sensible defaults and only requires a small amount of modification out of the box.

## Requirements

On the Ansible controller:
* The `netaddr` python package
* The `unzip` system package found on most Linux distributions

For the target hosts/environment:
* Linux
* `systemd`

## Dependencies

If you do not want to use consul-template in this set-up at all, then there are no dependencies!

The consul-template (`ctpl`) tasks in `tasks/ctpl.yml` will require the installation and configuration of consul-template as an instantiated `systemd` service  - my [Ansible role for consul-template](https://github.com/brickstool/ansible-role-consul-template) will do this for you.

Otherwise if you would like to configure this yourself, the `systemd` unit file for the consul-template service should look something like this:

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

Notice the use of `%I` in the example?
This allows for dynamically specifying a configuration file located in `/etc/consul-template.d` when starting the service.

Using the example configuration file in this role (`templates/ctpl.consul.hcl.j2`) along with the `systemd` unit file block above, you would execute the following command to enable and start the service:

```sh
$ sudo systemctl enable --now consul-template@consul.service
```

Using consul-template with the templates provided will also require a working HashiCorp Vault cluster with the PKI secrets engine enabled.
This is absolutely outside the scope of this role, sorry.

## Example Playbook

The following examples are the minimum configuration you would need to successfully run this role.
Generate a different encryption key for `consul_encrypt_string` using `consul keygen`, replacing "`NlJcajOKaGiitpFQBLA7BlDlu25PSm3AkRUYAI2MixE=`" with the generated key in the examples below.

*For a Consul server node*:

```yaml
- hosts: consul-server
  become: yes
  roles:
    - role: brickstool.consul
      vars:
        consul_server: true
        consul_encrypt_string: 'NlJcajOKaGiitpFQBLA7BlDlu25PSm3AkRUYAI2MixE='
        consul_retry_join:
          - 10.0.0.1
```

*For a Consul client node*:

```yaml
- hosts: consul-client
  become: yes
  roles:
    - role: brickstool.consul
      vars:
        consul_encrypt_string: 'NlJcajOKaGiitpFQBLA7BlDlu25PSm3AkRUYAI2MixE='
        consul_retry_join:
          - 10.0.0.1
```

## License

MIT / BSD

## Author Information

Created by Samuel Noordhuis in 2020. Inspired heavily by the Ansible roles and writings of [Jeff Geerling](https://github.com/geerlingguy).

If you see any errors or think this role could be improved in some way, you are welcome to open an issue/feature request or create a pull request :)

## TO-DO

1. Add additional configuration options for Consul
