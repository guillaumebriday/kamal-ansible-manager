# Kamal Ansible Manager

This is an [Ansible](https://www.ansible.com/) playbook to automatically optimize and secure your servers for [Kamal](https://kamal-deploy.org/), for Ubuntu only.

## What's inside?

It will automatically update your packages and configure these packages to secure your server(s):

- [Docker](https://docs.docker.com/engine/install/ubuntu/)
- [Fail2ban](https://github.com/fail2ban/fail2ban)
- [UFW](https://wiki.ubuntu.com/UncomplicatedFirewall)
- [NTP](https://ubuntu.com/server/docs/network-ntp)

The playbook also:

- Removes [Snap](https://snapcraft.io/).
- Disables ssh password login.

## Getting Started

Clone the repo:

```bash
git clone git@github.com:guillaumebriday/kamal-ansible-manager.git
cd kamal-ansible-manager
```

Copy the example files:

```bash
cp hosts.ini.example hosts.ini
cp ansible.cfg.example ansible.cfg
```

Update the `<host1>` in `hosts.ini` with your server's IP address (you can have multiple servers):

Run the playbook:

```bash
ANSIBLE_HOST_KEY_CHECKING=False ansible-playbook -i hosts.ini playbook.yml
```

## Contributing

Do not hesitate to contribute to the project by adapting or adding features! Bug reports or pull requests are welcome.

## License

This project is released under the [MIT](http://opensource.org/licenses/MIT) license.
