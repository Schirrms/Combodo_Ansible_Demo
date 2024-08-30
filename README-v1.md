# Combodo Ansible Demo

Some Ansible Demo with the Combodo Ansible Roles

## Goals

As I converted the combodo Ansible playbook `itop.core.write.yml` as an Ansible role (The repo is here : [Combodo Ansible](https://github.com/Schirrms/combodo_ansible) ), I thought I might as well show some examples of "simple" use cases here.

Bear in mind that, although directly usable, those are just examples, to hopefully helps you to create your own workflows.

Also, all examples here have been tested on a fresh new iTop 3.1.1 installation, using the iTop demo dataset, without any extension (you don't need any iTop extension to use the `itop.core.write.yml` Ansible playbook).

New : as I added two new roles in the [Combodo Ansible](https://github.com/Schirrms/combodo_ansible) repo, there are also two new demo in this folder :

To test the `itop_core_ispresent` role, you can use the demo `is_osversion_present.yml` (More info [below](#reading-the-number-of-a-specific-os-version) )

To test the `itop_core_read` role, you can use the demo `display_hypervisor.yml` (More info [below](#show-information-about-one-hypervisor))

To test the `itop_core_delete` role, you can use the demo `delete.virtualmachine.yml` (More info [below](#delete-a-virtual-machine))

## Usage

### Setup

Prerequisite: having a account member of `Administrator` and `REST Services User` profiles. For my demo, I use the `admin` account.

Download the repo, and setup your var in the file `host_vars/localhost.yml`

As the samples use the Combodo's Ansible role, you have to install a local copy of this role. This has to be done once.

The simplest way is trough then `ansible-galaxy` command:

~~~bash
ansible-galaxy collection install -r collections/requirements.yml
~~~

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
vm_osfamily: "Ubuntu server"
vm_osversion: "22.04 LTS"
vm_cpu: "2"
vm_ram: "1024"
vm_move2prod: "2024-02-08"
vm_interfaces: 
  - { "name":"ens192", "mac":"ef:34:56:78:90:ab", "speed": "10000" }
  - { "name":"eth0", "mac":"11:22:33:44:55:66", "speed": "1000" }
vm_contacts:
  - "claude.monet@demo.com"
  - "anna.gavalda@it.com"
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
localhost                  : ok=100  changed=5    unreachable=0    failed=0    skipped=60   rescued=0    ignored=0
~~~

And one Virtual Machine is created.

As expected with Ansible, if you run the same playbook again:

~~~bash
ansible-playbook -e "@files/VM_Ansible_Test.yml" create-update.virtualmachine.yml
[WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost does not match 'all'

PLAY [Create a Virtual Machine in iTop] ************************************************************************************
………………………
TASK [schirrms.combodo_ansible.itop_core_write : Get fields] ***************************************************************
ok: [localhost]

PLAY RECAP *****************************************************************************************************************
localhost                  : ok=105  changed=0    unreachable=0    failed=0    skipped=55   rescued=0    ignored=0
~~~

No changes where done in iTop.

### Reading the number of a specific OS Version

This is a very simple example. The OSVersion name is set in the playbook so you don't need any external files.
Remember that the expected results are on a clean installation of the Combodo's demo set.

*** Warning : even if the returning value, `itop_found_items` seems realistically a number, Ansible's set_fact stores the value as a string.
So, if you want to know if a CI is present, the correct test would be:

~~~yaml
    when: itop_found_items|int == 1
~~~

if `osversion: '22.04 LTS'` then, launching this command:

~~~bash
ansible-playbook is_osversion_present.yml
[WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost does not match 'all'

PLAY [Query the number of 'OSVersion' 22.04 LTS] *************************************************************************
…………………

TASK [Display the result] ************************************************************************************************
ok: [localhost] => {
    "msg": "the OSVersion '22.04 LTS' was found 2 time(s)"
}

PLAY RECAP ***************************************************************************************************************
localhost                  : ok=8    changed=0    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0  
~~~

Of course, instead of editing the file, you can also set the osversion on command line, so :

~~~bash
ansible-playbook -e "osversion='11.5'" is_osversion_present.yml
[WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost does not match 'all'

PLAY [Query the number of 'OSVersion' 11.5] *******************************************************************************
…………………
TASK [Display the result] *************************************************************************************************
ok: [localhost] => {
    "msg": "the OSVersion '11.5' was found 1 time(s)"
}

PLAY RECAP ****************************************************************************************************************
localhost                  : ok=8    changed=0    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0 
~~~

### Show information about one hypervisor

Again a very simple example. The hypervisor name is set in the playbook so you don't need any external files.
Remember that the expected results are on a clean installation of the Combodo's demo set.

With `hypervisor: 'ESX1'` you can run the script this way:

~~~bash
ansible-playbook display_hypervisor.yml
[WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost does not match 'all'

PLAY [Queries hypervisors in iTop] ***************************************************************************************
…………………
TASK [Display Hypervisor values] ******************************************************************************************
ok: [localhost] => {
    "itop_output_values": [
        {
            "name": "ESX1",
            "organization_name": "Demo",
            "server_name": "Server1",
            "status": "production"
        }
    ]
}

TASK [Display Hypervisor 0 Server name] ***********************************************************************************
ok: [localhost] => {
    "itop_output_values[0].server_name": "Server1"
}

PLAY RECAP ****************************************************************************************************************
localhost                  : ok=9    changed=0    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0
~~~

Setting another value in the file for hypervisor, or changing the value on the command line:

~~~bash
ansible-playbook display_hypervisor.yml -e 'hypervisor="inexistant_hypervisor"'
[WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost does not match 'all'

PLAY [Queries hypervisors in iTop] ****************************************************************************************
……………………
TASK [Display Hypervisor values] ******************************************************************************************
ok: [localhost] => {
    "itop_output_values": ""
}

TASK [Display Hypervisor 0 Server name] ***********************************************************************************
skipping: [localhost]

PLAY RECAP ****************************************************************************************************************
localhost                  : ok=8    changed=0    unreachable=0    failed=0    skipped=3    rescued=0    ignored=0
~~~

### Display all Hypervisors

Alternatively, you can also retrieve a complete class, with a little twist on the 'itop_key' value, that should be set to 'SELECT {ClassName}' (Or a filtered set if you put a OQL query).

You'll then have to parse the `itop_output_values` array

As an example, I created the `display_all_hypervisors.yml` playbook:

~~~bash
ansible-playbook display_all_hypervisors.yml 
[WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost does not match 'all'

PLAY [Queries hypervisors in iTop] ****************************************************************************************
……………………
TASK [Display Hypervisors values] *****************************************************************************************
ok: [localhost] => {
    "itop_output_values": [
        {
            "name": "ESX1",
            "organization_name": "Demo",
            "server_name": "Server1",
            "status": "production"
        },
        {
            "name": "ESX2",
            "organization_name": "Demo",
            "server_name": "Server3",
            "status": "production"
        },
        {
            "name": "ESX3",
            "organization_name": "Demo",
            "server_name": "",
            "status": "production"
        }
    ]
}

PLAY RECAP *************************************************************************************************************
localhost                  : ok=8    changed=0    unreachable=0    failed=0    skipped=2    rescued=0    ignored=0 
~~~

### Delete a Virtual Machine

***Warning : the role behind the deletion, `itop_core_delete` is still in an early stage for now. While it only call the corresponding well defined `core/delete` Combodo's API, I cannot say that this is bulletproof. Use at your own risk!***

This role is the result of my own work, please do not ask Combodo's help!

Again a very simple example. This time, we will remove the VirtualMachine that we builded as the first example.

We will need some information about the Virtual Machine, fortunately, those information are already available in the configuration file VM_Ansible_Test.yml

So, to suppress the virtual machine, you only have to launch:

~~~bash
ansible-playbook -e "@files/VM_Ansible_Test.yml" delete.virtualmachine.yml
[WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost does not match 'all'

PLAY [Delete a Virtual Machine in iTop] ***********************************************************************************
……………………………
TASK [Display deletion result] ********************************************************************************************
ok: [localhost] => {
    "msg": "Deletion return code: '0', return message: 'Deleted: 1 plus (for DB integrity) 4'"
}

PLAY RECAP ****************************************************************************************************************
localhost                  : ok=12   changed=1    unreachable=0    failed=0    skipped=5    rescued=0    ignored=0   
~~~

Of course, if the VirtualMachine doesn't exist, then there is no changes:

~~~bash
ansible-playbook -e "@files/VM_Ansible_Test.yml" delete.virtualmachine.yml
[WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost does not match 'all'

PLAY [Delete a Virtual Machine in iTop] ***********************************************************************************
……………………………
TASK [Display deletion result] ********************************************************************************************
ok: [localhost] => {
    "msg": "Deletion return code: '0', return message: 'Found: 0'"
}

PLAY RECAP ****************************************************************************************************************
localhost                  : ok=9    changed=0    unreachable=0    failed=0    skipped=8    rescued=0    ignored=0   
~~~
