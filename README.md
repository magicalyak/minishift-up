[![Build Status](https://travis-ci.org/magicalyak/minishift-up.svg?branch=master)](https://travis-ci.org/magicalyak/minishift-up)

# minishift-up

Installs the latest [minishift](https://github.com/minishift/minishift) binary, and creates a minishift instance.

Performs the following tasks:

- Downloads and installs the latest minishift binary
- Copies the latest oc binary from ~/.minishift/cache/oc to a directory in your PATH
- Installs the Docker Machine driver
- Creates a minishift instance
- Grants cluster admin to the *admin* account

## Accessing the registry

After the role executes, and minishift is running, you will be able to access the internal registry after setting the minishift docker-env eval command:

```sh
    eval $(minishift docker-env)
    docker login -u developer -p $(oc whoami -t)
```

## Supported platforms

- Debian
- Red Hat
- OSX

## Prerequisites

- Ansible 2.4+
- Prior to running the role, clear your terminal session of any DOCKER* environment variables.
- sudo access is required for installing packages

### OSX

- [homebrew](https://brew.sh)
- [Ansible 2.4+](https://docs.ansible.com)

#### Mounting /Users to the Minishift VM

When the Minishift VM is started, the /Users volume will be mounted to the VM. This is done by setting the environment variable `XHYVE_VIRTIO_9P=true`.

## Linux

- KVM installed and working. The role installs the Docker Machine driver for KVM, but it assumes KVM is already installed, and working.
- Ansible 2.4+

## Fedora

- Install packages python2-dnf, and libselinux-python

## Known Issues

### Fedora

- Before accessing the Docker daemon on the Minishift instance, you'll need to modify the `/etc/sysconfig/docker` script to prevent it from overriding the DOCKER_CERT_PATH environment variable. Edit the file, and change the line `DOCKER_CERT_PATH=/etc/docker` to the following:

    ```sh
    if [ -z "${DOCKER_CERT_PATH}" ]; then
        DOCKER_CERT_PATH=/etc/docker
    fi
    ```

## Defaults

**minishift_repo:** minishift/minishift

> Repo where the minishift binary can be found

**minishift_github_url:** <https://api.github.com/repos>

> URL to access GitHub API.

**minishift_release_tag_name:** ""

> Defaults to installing the latest release. Use to install a specific minishift release.

**minishift_dest:** /usr/local/bin**

> Where to install the minishift binary.

**minishift_force_install:** yes

> Overwrite any existing minishift binary found at {{ minishift_dest }}

**minishift_restart:** yes

> Stop and recreate the existing minishift instance.

**minishift_delete:** yes

> Perform `minishift delete`, and remove `~/.minishift`. If you're upgrading, you most likely want to do this.

**minishift_start_options: []**

> Provide a list of options to pass to `minishift start`. For example: `['--memory', '4GB', '--cpus', '4']`

**openshift_client_dest:** /usr/local/bin

> Where to install the OpenShift client binary.

**openshift_force_client_copy:** yes

> Overwrite any existing OpenShift client binary found at {{ openshift_client_dest }}.

**use_hyperkit:** no

> For MaxOSX you can "try to" use hyperkit instad of xhyve by setting this to yes

## Example Playbook

Below is a sample playbook that includes all of the default parameters. You'll find this exact example in the root of the project tree.

```yaml
---
- name: Install minishift
  hosts: localhost
  connection: local
  gather_facts: yes
  roles:
    - role: magicalyak.minishift_nginx
      minishift_repo: minishift/minishift
      minishift_github_url: https://api.github.com/repos
      minishift_release_tag_name: ""
      minishift_dest: /usr/local/bin  
      minishift_force_install: yes
      minishift_restart: yes
      minishift_delete: yes
      minishift_start_options: []
      openshift_client_dest: /usr/local/bin
      openshift_force_client_copy: yes
      use_hyperkit: no
```

After you install the role, copy *minishift-nginx.yml* to your project directory, and execute it with the `--ask-become-pass` option. For example:

```sh
# Install the role
$ ansible-galaxy install magicalyak.minishift_up

# Copy the playbook from your roles path to the current working directory
$ cp ${ANSIBLE_ROLES_PATH}/magicalyak.minishift_up/minishift-up.yml .

# Create a localhost inventory file
$ echo "localhost">./inventory

# Run the playbook
$ ansible-playbook -i inventory --ask-become-pass minishift-up.yml
```

## Dependencies

none

## License

Apache v2

## Author

[@magicalyak](https://github.com/magicalyak)

## Credits

I took most of this off of Chris Houseknechts role
[minishift-up-role](https://galaxy.ansible.com/chouseknecht/minishift) on ansible galaxy
[@chouseknecht](https://github.com/chouseknecht)
