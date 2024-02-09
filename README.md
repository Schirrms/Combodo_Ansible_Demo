# Combodo Ansible Demo

Some Ansible Demo with the Combodo Ansible Role

## Goals

As I converted the combodo Ansible playbook `itop.core.write.yml` as an Ansible role (The repo is here : [Combodo Ansible](https://github.com/Schirrms/combodo_ansible) ), I thought I could also put here some 'simple' case usage examples.

Bear in mind that, although directly usable, those are just examples, to hopefully helps you to create your own workflows.

Also, all examples here have been tested on a full new iTop 3.1.1 installation, using the iTop demo dataset, without any extension (you don't need any iTop extension to use the `itop.core.write.yml` Ansible playbook).

## Usage

### Setup

Prerequisite: having a account member of 'Administrator' and 'REST Services User' profiles. For my demo, I use the admin account.

Download the repo, and setup your var in the file `host_vars/localhost.yml`
There is a sample file (coming directly from the original playbook): `host_vars/localhost_yml.sample`. There are to many information here for now, but you should set up as a bare minimal:

~~~yaml
# URL to reach iTop web services
itop_root: {{ Main Url of your iTop Instance }}
itop_ws_version: 1.3

# Connection parameters
# If you prefer to encrypt these parameter in an Ansible vault for security reasons:
#  - Comment the following 2 lines
#  - Store the parameter in a local itopvault.yml file
#  You may refer to Ansible documentation: https://docs.ansible.com/ansible/latest/vault_guide/index.html
itop_ws_auth_user: {{ your iTop account with 'Admin' and 'Rest Service User' profile }}
itop_ws_auth_pwd: {{ password for this account }}

# Although not really used at this point, the two next values are meant to be presents. Let the default value if you don't have installed and configured the companion iTop extension 'Data model for Ansible' (Available on the iTop Hub)
# UUID in iTop of the Ansible application to work with
itop_ws_ansible_uuid: DEMO_UUID

# Name of the inventory to consider
itop_ws_ansible_inventory: DMZ-1
~~~

Bear in mind that this is a very unsecure way of configuring an admin access to your iTop instance! You should at a bare minimum use the vault encryption for this file, or maybe use as proposed by Combodo a separate file for your credentials.

### Creating or Updating information about a Virtual Machine in iTop

The playbook is `create-update.virtualmachine.yml`. This playbook needs some information about the VM creation/update.

For the sake of this example, I stored a set of vars in the file `files/VM_Ansible_Test.yml`:

~~~yaml
org_name: "Demo"
vm_name: "Demo_Ansible_Test_VM"
vm_ip: "10.64.65.169"
vm_host: "Cluster1"
vm_osfamily: "Oracle Linux"
vm_osversion: "18.04 LTS"
vm_cpu: "1"
vm_ram: "1024"
vm_move2prod: "2024-02-08"
vm_int_name: "ens192"
vm_int_mac: "ef:34:56:78:90:ab"
vm_int_speed: "10000"
~~~

With this file, you can then run this command:

~~~bash
ansible-playbook -e "@files/VM_Ansible_Test.yml" create-update.virtualmachine.yml
[WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost does not match 'all'

PLAY [Create a Virtual Machine in iTop] ************************************************************************************
………………………
TASK [schirrms.combodo_ansible.itop_core_write : Get fields] ***************************************************************
ok: [localhost]

PLAY RECAP *****************************************************************************************************************
localhost                  : ok=38   changed=2    unreachable=0    failed=0    skipped=22   rescued=0    ignored=0
~~~

And one Virtual Machine is created.

As expected in Ansible, if you run the same playbook again:

~~~bash
ansible-playbook -e "@files/VM_Ansible_Test.yml" create-update.virtualmachine.yml
[WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost does not match 'all'

PLAY [Create a Virtual Machine in iTop] ************************************************************************************
………………………
TASK [schirrms.combodo_ansible.itop_core_write : Get fields] ***************************************************************
ok: [localhost]

PLAY RECAP *****************************************************************************************************************
localhost                  : ok=40   changed=0    unreachable=0    failed=0    skipped=22   rescued=0    ignored=0
~~~

No changes where done in iTop.
