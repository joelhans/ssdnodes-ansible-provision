# ssdnodes-ansible-provision

A basic Ansible playbook to provision bare servers with SSH keys, iptables, and fail2ban.

This playbook was created in conjunction with two blog posts written for the [SSD Nodes blog](http://blog.ssdnodes.com/blog).

[Tutorial: A More Secure Ansible Playbook, Part 1](http://blog.ssdnodes.com/blog/tutorial-a-more-secure-ansible-playbook-part-1)

[Tutorial: A More Secure Ansible Playbook, Part 2](https://blog.ssdnodes.com/blog/tutorial-a-more-secure-ansible-playbook-part-2/)

This playbook was built as an educational tool for the SSD Nodes blog. It **is not an officially supported tool**, and comes with no support other than these instructions and any discussions that might be found in the [repository issues](https://github.com/joelhans/ssdnodes-ansible-provision/issues).

## Prerequisites

A bare/newly-created VPS running **Ubuntu 16.04**.

Debian 8 may work out-of-the-box as well. CentOS is not currently supported, but will be in the near future.

## Instructions

**Clone the repository**.

```
$ git clone https://github.com/joelhans/ssdnodes-ansible-provision.git
```

**Generate a hashed password**.

On Linux/OS X machines, you can use the following method to create a hashed password.

First, you need to install the Passlib password hashing library for Python, if you don't have it already.

```
$ pip install passlib
```

Once Passlib is installed, run the following command **after** replacing `password` with the phrase of your choosing.

```
$ python -c 'from passlib.hash import sha512_crypt; print sha512_crypt.encrypt("password")'
```

Once you have the hashed password, you can copy it into the `vars:password` field in `provision.yml`.

**Edit additional variables in `provision.yml`.**

You need to specify which hosts you would like to target by changing the `server` field in the `hosts` setting. See [Ansible docs](http://docs.ansible.com/ansible/latest/intro_inventory.html) for more information about setting your hosts and groups.

You should also change the `vars:username` variable to the non-root user account you would like to create.

If your SSH key is not in the default location—`~/.ssh/id_rsa.pub`—you will need to change that as well.

**Run the playbook.**

```
$ ansible-playbook -k provision.yml
```

If you run into an error about `/usr/bin/python` not being found, you need to force Ansible to use `python3` on the server by adding the following flag to your Ansible hosts file:

```
[example]
123.123.123.123 ansible_python_interpreter=/usr/bin/python3
```

### Re-running the playbook

Because the tasks in this playbook are idempotent, you can run this playbook any number of times you would like, making small tweaks. But, after the first run, the default configuration will fail because Ansible won't be able to make an SSH connection—we disabled root logins via `/etc/ssh/sshd_config`, remember?

In order for Ansible to connect to the server after the initial run, you need to change two settings within `provision.yml`. Here's what the file looks like by default:

```
remote_user: root
# become: true
```

The `remote_user` setting needs to be changed to match up with with your non-root user—the one you specified in `vars:username`. You also need to uncomment the `become: true` line, which allows this non-root user to run all operations with sudo. Here's what it should look like after the fact.

```
remote_user: YOUR-USER
become: true
```

Now that the settings are changed, you can re-run the playbook:

```
$ ansible-playbook -k provision.yml --ask-become-pass
```

Ansible will ask for the SSH password, which is the one you hashed earlier. It will then ask for the sudo password, which is the same, so you can either re-enter the password or hit `Enter`. The playbook will then run as normal.

## License

This playbook is licensed under MIT.
