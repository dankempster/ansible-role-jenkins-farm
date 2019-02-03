# Ansible Role: Configure a Jenkins Build farm

Deploys a Jenkins build farm

## Dependencies

This role doesn't actually install Jenkins. It's designed to work with these roles that do:

 - [geerlingguy.jenkins](https://github/geerlingguy/ansible-role-jenkins) to install Jenkins
 - [dankempster.jenkins-config](https://github/dankempster/ansible-role-jenkins-config) for configuring Jenkins


## Role Variables

Available variables are listed below, along with default values (see `defaults/main.yml`):

```yaml
# inventory name of the master node for the farm
jenkins_farm_master: ""
```

Declare which host (from inventory) is the master Jenkins node.


```yaml
# the hostname to access Jenkins' via
jenkins_hostname: "{{ jenkins_farm_master }}"
```

The hostname to use for Jenkins HTTP address, defaults to the master node's hostname.

_Note: `jenkins_hostname` was originally defined in [geerlingguy.jenkins](https://github/geerlingguy/ansible-role-jenkins)._


```yaml
# passphrase set for Jenkins' SSH private key
jenkins_farm_ssh_passphrase: "secret"
```

The passphrase used when generating Jenkins' SSH key pair, which will be used for connecting to the slave nodes.


```yaml
# SSH user for Jenkins to use when connecting to the slaves
jenkins_farm_ssh_user: "{{ jenkins_process_user }}"
jenkins_farm_ssh_group: "{{ jenkins_process_group }}"
jenkins_farm_home: "/home/{{ jenkins_farm_ssh_user }}"
```

The user, group and home directory to create on each slave.
Defaults to the same user and group as on the master, defined by `jenkins_process_user` & `jenkins_process_group` from [geerlingguy.jenkins](https://github/geerlingguy/ansible-role-jenkins) role.


```yaml
# password for your use when SSHing as Jenkins' user
#   should be in crypto format, this example crypto text is 'secret'
jenkins_farm_ssh_password: "$6$xJ5H4uQhfxcL3nY7$Tam0bIhAigVy8/6V88ppCGuGSJB83GW\
  ce/zTAHnmDvMvVgb3MKrrChW0LhrtciaraQuYJGZSbHoRzb.qbAP7.."
```

The SSH password set for the jenkins user on each slave. This is not used by the role itself, it's simply to allow you to log in.


```yaml
# the role of the current node (worked out automatically)
jenkins_farm_role: "{{
  (inventory_hostname == jenkins_farm_master) | ternary('master', 'slave')
  }}"
```

The role of each node, can be either `master` or `node`. There should only be one `master` per farm. 
This is calculated automatically based on `jenkins_farm_master`. Generally, you shouldn't need to change it.


## Example Playbook

Following is a simple playbook that, depending on inventory, can be used in several ways.

First, you'll need this simple playbook:

```yaml
# playbook.yml

- hosts: jenkins_farm
  roles:
    - role: roles/geerlingguy.java
      become: true

    - role: roles/geerlingguy.jenkins
      become: true
      when: jenkins_farm_role == 'master'

    - role: roles/dankempster.jenkins-config

```


### A single farm

Following is an example `inventory` file for setting up a simple Jenkins build farm.

```yaml
# inventory.yml

jenkins_farm:
  hosts:
  	jenkins_master.example.com:
    jenkins_slave1.example.com:
    jenkins_slave2.example.com:
  vars:
    jenkins_farm_master: jenkins_master.example.com

```


### Config groups

Set different configuration to fit various nodes hardware and OSes.

```yaml
# inventory.yml

jenkins_raspberrypis:
  hosts:
    rpi1.example.com:
    rpi2.example.com:

jenkins_amd64:
  hosts:
    amd64.example.com:

jenkins_master:
  hosts:
    jenkins_master.example.com
    
jenkins_farm:
  children:
    jenkins_master:
    jenkins_raspberrypis:
    jenkins_amd64:
  vars:
    jenkins_farm_master: jenkins_master.example.com

```

Now, you can configure the master and slaves to suite their resources

```yaml
# group_vars/jenkins_master.yml

# don't tie up the master building stuff itself
jenkins_config_executors: 0

```

```yaml
# group_vars/jenkins_raspberrypis.yml

jenkins_config_executors: 1

jenkins_labels:
  - arm
  - linux
  - rasberrypi

```

```yaml
# group_vars/jenkins_amd64.yml

jenkins_config_executors: 2

jenkins_labels:
  - amd64
  - linux

```


### Multiple Farms

It's easy to configure multiple farms simultaneously by grouping the nodes of each farm together and define the `jenkins_farm_master` for each group.

```yaml
# inventory.yml

# the development farm
dev_farm:
  hosts:
  	master.dev.farm.example.com:
    slave1.dev.farm.example.com:
    slave2.dev.farm.example.com:
  vars:
    jenkins_farm_master: master.dev.farm.example.com

# the QA build farm
qa_farm:
  hosts:
  	master.qa.farm.example.com:
    slave1.qa.farm.example.com:
    slave2.qa.farm.example.com:
  vars:
    jenkins_farm_master: master.qa.farm.example.com

jenkins_farm:
  children:
    dev_farm:
    qa_farm:

```


## License

MIT

## Author Information

This role was created in 2019 by [Dan Kempster](https://github.com/dankempster).
