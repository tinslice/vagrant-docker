# vagrant-test-env

Vagrant configuration for test virtual machines.

## Usage

Create a `.vagrantuser` file for custom configuration as in the following example:

```yml
vm_box: ubuntu/bionic64
# vm_name: my-custom-domain
## Show VM Interface
# show_gui: true
ssh:
  ## Enable ssh authentication with custom ssh key
  # public_key: "~/.ssh/id_rsa.pub"
nodes:
  - name: node-01
    cpu: 1
    memory: 1024
    ip: 192.168.90.21
  - name: node-02
    cpu: 1
    memory: 1024
    ip: 192.168.90.22
docker:
  ## Enable docker install
  # install: true
  # registry: your-private-registry
  # user:
  # password:
```

**Install the 'nugrant' plugin**

```bash
vagrant plugin install nugrant
```

**Auto update hosts file**

```bash
vagrant plugin install vagrant-hostsupdater
```

**Start the virtual machines**

```bash
vagrant up
```

NOTE: the first time you start the vm vagrant will ask to install the required plugins. After the install completes you will need to run the command again.

**SSH into the VM**

```bash
# vagrant ssh <node-name>
vagrant ssh node-01
```

**Other commands**

```bash
# stop all virtual machines
vagrant halt
# restart all virtual machines
vagrant reload
```