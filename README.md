# Kamal Ansible Manager

This is an [Ansible](https://www.ansible.com/) playbook to automatically optimize and secure your servers for [Kamal](https://kamal-deploy.org/), for Ubuntu only.

Here's a quick video walkthrough of how this repo works:

[![Watch the video](https://raw.githubusercontent.com/guillaumebriday/kamal-ansible-manager/main/thumbnail.png)](https://www.youtube.com/watch?v=WTYNPCEJSOo)

## What's inside?

It will automatically update your packages and configure these packages to secure your server(s):

- [Docker](https://docs.docker.com/engine/install/ubuntu/)
- [Fail2ban](https://github.com/fail2ban/fail2ban)
- [UFW](https://wiki.ubuntu.com/UncomplicatedFirewall)
- [NTP](https://ubuntu.com/server/docs/network-ntp)

The playbook also:
- Remove [Snap](https://snapcraft.io/).
- Disable ssh password login.
- Configure `swap` using [geerlingguy/ansible-role-swap](https://github.com/geerlingguy/ansible-role-swap).

## Getting Started

Clone the repo:
```bash
$ git clone git@github.com:guillaumebriday/kamal-ansible-manager.git
$ cd kamal-ansible-manager
```

Copy the inventory example file:
```bash
$ cp hosts.ini.example hosts.ini
```

Update the `<host1>` with your server's IP address (you can have multiple servers):
```bash
$ vim hosts.ini
```

Install the requirements:
```bash
$ ansible-galaxy install -r requirements.yml
```

## Configuring vars

Variables can be configured in the `playbook.yml` file.
Also, you can override default variables provided in [geerlingguy/ansible-role-swap](https://github.com/geerlingguy/ansible-role-swap/blob/master/defaults/main.yml) to adjust the swap settings.

For instance:
```yml
  vars:
    security_autoupdate_reboot: "true"
    security_autoupdate_reboot_time: "03:00"
    swap_file_size_mb: '1024'
```

## Running the playbook

Run the playbook:
```bash
$ ANSIBLE_HOST_KEY_CHECKING=False ansible-playbook -i hosts.ini playbook.yml
```

## Contributing

Do not hesitate to contribute to the project by adapting or adding features ! Bug reports or pull requests are welcome.

## License

This project is released under the [MIT](http://opensource.org/licenses/MIT) license.
