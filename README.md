# Ansible Role: ProtonMail Bridge (headless)

Installs the ProtonMail Bridge (optionally prompting for 2FA), registers it as a headless service, and configures Postfix to use it. Supports RedHat/CentOS,
Debian/Ubuntu, and Archlinux servers.

## Background
The [ProtonMail Bridge app for Linux](https://protonmail.com/support/knowledge-base/bridge-for-linux/) enables you to integrate 
your ProtonMail account with IMAP and SMTP email programs such as Thunderbird and Evolution. Bridge is available to all 
ProtonMail users with a paid subscription.

The purpose of this role is to automate the bridge installation such that it can be used on a headless machine without
a GUI.

Specifically, this role will:

* download the latest version of the bridge
* ensure the package is correctly signed before installing it
* configure the bridge with your user information
* create a service to keep the bridge running in the background
* create a locked user to run the service as
* configure postfix for use with the bridge (SASL)
* on RHEL systems, (or optionally) configures SELinux contexts

## Requirements

Requires RedHat/CentOS 8, Ubuntu 18 or higher, or an up-to-date Debian, Archlinux, or Manjaro system. 

## Role Variables

Available variables are listed below, along with default values (see `defaults/main.yml`):

    protonmail_username: ""
    protonmail_password: ""
    protonmail_custom_domain: ""
    protonmail_enable_2fa: false

Your ProtonMail credentials and domain. These are used to add your account to the ProtonMail-bridge, and to configure 
postfix. __NOTE__: when `protonmail_enable_2fa` is set to `true`, this role will be run interactively and will prompt you 
for a 2FA code.

    # configure_selinux: ""

Whether to configure SELinux. Defaults to true on RHEL systems and false otherwise.

    gpg_key_settings:
        type: default
        length: default
        subkey_type: default
        subkey_length: default
        expire_date: 0
        name: protonmail-bridge-headless service key
        email: root@localhost

These settings are used to create a local GPG key. The key is required for the ProtonMail bridge and `pass` to 
communicate. Additional changes can be made to the `templates/protonmail-bridge.gpg.j2` file. _NOTE: the default 
settings generate a new unprotected key to use exclusively for the bridge. To add a key password, you will need to 
modify the default values, as well as `templates/protonmail-bridge-headless.service.sh.j2`._

    configure_gpg: "true"
    configure_pass: "true"
    configure_postfix: "true"
    configure_service: "true"
    configure_user: "true"

Can be used to skip tasks in this role.

    protonmail_lib_dir: "/var/lib/protonmail"

The location where configuration and service files will be stored.

    protonmail_user: "protonmail"
    protonmail_user_flags: "-L"

The name and flags of the service-user.

    postfix_hostname: "{{ protonmail_custom_domain }}"
    postfix_localhost_address: "127.0.0.1"

Settings you can override in `main.cf`.

    selinux_httpd_can_sendmail: "true"

Whether to add selinux policies for httpd when configuring postfix.

## Dependencies

A paid ProtonMail account.

## Sample usage

First, install a current version of [Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#installing-ansible-on-specific-operating-systems):  
Archlinux:
``` 
sudo pacman -Syu ansible
```
RHEL:
```
sudo dnf -y install epel-release
sudo dnf repolist
sudo dnf -y install ansible
```
Debian:
```
sudo apt update
sudo apt install -y software-properties-common
sudo apt-add-repository -y -u ppa:ansible/ansible
sudo apt install -y ansible
```

Then, create a file called `main.yml` and add the below (replacing `your_username`, `your_password`, and `your_domain` 
with their actual values:

    - name: "install and configure protonmail-bridge as a headless service"
      become: "yes"
      hosts: "all"
      vars:
        protonmail_username: "your_username"
        protonmail_password: "your_password"
        protonmail_custom_domain: "your_domain"
      roles:
        - "moismailzai.protonmail_bridge_headless"

Then from the command line in the same directory, run:

```
# installs this role from Ansible Galaxy
ansible-galaxy install moismailzai.protonmail_bridge_headless

# ensures Ansible can find the role you just downloaded
ln -s ~/.ansible/roles roles

# runs the playbook you created in the previous step
sudo ansible-playbook -c local -i localhost, main.yml
```

Once the process is complete, send yourself a test message from the command line: 

```
echo "If you are seeing this, the bridge has been correctly configured." | mail -s "ProtonMail-bridge test message" -r your@email.address recipients@email.address
```

## License

MIT

## Author Information

This role was created in 2021 by [Mo Ismailzai](https://www.ismailzai.com/).
