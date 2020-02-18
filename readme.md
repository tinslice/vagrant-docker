# vagrant-docker

Vagrant configuration for docker enabled virtual machine.

NOTE: 

- The Vagrant configuration exposes some ports so if you what to remove them just add a `#` in front of each `forwarded_port` rule.
- The mount location is `/workspace`

## Usage

Create a `.vagrantuser` for custom configuration as in the following example bellow:

```yml
services:
  vm_name: docker-dev
  config_location: path/to/your/project/or/docker-compose/configuration
docker:
  registry: docker-registry-domain
  user: docker-user
  password: docker-password
```

**Start the virtual machine**

```bash
vagrant up
```

NOTE: the first time you start the vm vagrant will ask to install the required plugins. After the install completes you will need to run the command again.

**SSH into the VM**

```bash
vagrant ssh
```
