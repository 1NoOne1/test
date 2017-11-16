# Dell PowerEdge 620/730 Servers – Update iDrac Firm Ware/Settings/Device Drivers using RACADM.

This will update Firmware/Device Drivers, change/modify iDrac/Bios Settings using Racadm command line tool on Dell PowerEdge 620 and 730xd models. We will connect to the local racadm which resides in iDrac and execute the required commands using the Ansible RAW module. We use HTTP repository share from which we will update the recommended updates and these updates were created in the form a catalog using Dell Repository Manager.  

## Getting Started

These instructions will get you a copy of the project up and running on your local machine for development and testing purposes. 

### Prerequisites

* You should a system/VM running linux with --  `CentOS Linux release 7.0.1406 (Core) or above available`.
* Install Python --version:  `python version = 2.7.5.`
* [Ansible](http://docs.ansible.com/intro_installation.html) should be installed --version:  `ansible 2.3.1.0`.
* Pip (Python package Index) should installed using `epel-release`. --version: `pip 8.1.2 from /usr/lib/python2.7/site-packages (python 2.7)`.
* Jinja2 should be installed using `pip install jija2`. --version: `Jinja2 (2.7.2)`.
* A `HTTP server` should be setup, running and should be accessible to iDrac hosts. This is the location where we host Dell FW updates as HTTP share from which iDrac hosts would be able to download the required Firmware/device drivers.
 
### Installing
Download/Clone the Repository and place the directory structure as shown above. We are following the directory structure that is mentioned in anisble best practices. Please follow the below links for more understanding on how ansible directory structure works:
1. http://docs.ansible.com/ansible/latest/playbooks_best_practices.html#directory-layout
2. https://leucos.github.io/ansible-files-layout


### Inventory Setup

`inventory\hosts` : We will configure the entire stack by listing our hosts in the 'hosts' inventory file, usually grouped by their purpose:
```
[all:children]
idrac_hosts
#r620_servers
#r730_servers

#[r620_servers]
#10.231.9.46 idrac_racname=r6c03-bmc model=620

#[r730_servers]
#10.231.9.40 idrac_racname=r6c12-bmc model=730

[idrac_hosts]
#r620_servers
#r730_servers
10.231.9.46 idrac_racname=r6c03-bmc model=620
10.231.9.39 idrac_racname=r6c11-bmc model=730

#[r620_servers:vars]
[idrac_hosts:vars]
ansible_ssh_pass=******
ansible_ssh_user=root
idrac_dns1=10.231.0.101
idrac_dns2=10.231.0.103
idrac_gateway=10.231.9.1
idrac_netmask=255.255.255.0
idrac_domainname=encore-oam.com
raid_force=false
catalog_http_share=10.231.7.155/DellRepo/072017
mgmt_vlanid=104
mgmt_gateway=10.7.20.1
mgmt_netmask=255.255.255.0


```

```
[all:children] => main parent group
idrac_hosts     --- 
#r620_servers      | --> host groups
#r730_servers   ---

#[r620_servers]                                               
#10.231.9.46 idrac_racname=r6c03-bmc model=620  | --> host in-line variables that can be used in playbooks (applicable to only this host).

[idrac_hosts]
10.231.9.46 idrac_racname=r6c03-bmc model=620

#[r620_servers:vars]     | --> variables that are defined for the host-group (applicable to all the hosts in this group). 
[idrac_hosts:vars]                   
ansible_ssh_pass=******
ansible_ssh_user=root
idrac_dns1=10.231.0.101
...
...
...
```
### Run the Playbook
*Once we are done with defining all the required variables we are now ready to execute our playbook on the hosts by running the following command from the terminal*

```
ansible-playbook -i inventory/hosts playbooks/deploy_server_roles.yml
```
* **deploy_server_roles.yml** `=>` which holds the following five roles that gets executed accordingly. If you don't want a role to be executed then simply comment out that line in the YAML file.

```
  roles:
#    - role: ../roles/Firmware_Updates
    - role: ../roles/Raid_R620
      when: '(raid_force | bool) and (model is defined and model == 620)'
#    - role: ../roles/Raid_R730
#      when: '(raid_force | bool) and (model is defined and model == 730)'
    - role: ../roles/iDrac_Settings
    - role: ../roles/iDrac_BIOS_Settings

```

 1. `../roles/Firmware_Updates` : => Compare against the HTTP catalog and run any updates if available.
 2. `../roles/Raid_R620`        : => Reset the RAID forcefully and re-create RAID-1 across first two disks. Runs only when `raid_force` is set to true in `inventory\hosts` [when: '(raid_force | bool) and (model is defined and model == 620)'].
 3. `../roles/Raid_R730`        : => Reset the RAID forcefully and re-create RAID-1 across first two disks and convert any SSDs into Non-RAID. Runs only when `raid_force` is set to true in `inventory\hosts` [when: '(raid_force | bool) and (model is defined and model == 730)'].       
 4. `../roles/iDrac_Settings`   : => Modify/Update the iDrac settings using the variable values mentioned in invenroty/hosts.
 5. `../roles/iDrac_BIOS_Settings` " => Modify/Update the BIOS settings using the variable values mentioned in invenroty/hosts.
 ```
+---------------------------------+--------------------------------------------------------------------------------------------+
| Role                            | Description                                                                                |
+=================================+============================================================================================+
| ../roles/Firmware_Updates       | Compare against the HTTP catalog and run any updates if available.                         |
+---------------------------------+--------------------------------------------------------------------------------------------+
| ../roles/Raid_R620              | Reset the RAID forcefully and re-create RAID-1 across first two disks.                     | 
|                                 | Runs only when `raid_force` is set to true in `inventory\hosts`                            |
|                                 | [when: '(raid_force | bool) and (model is defined and model == 620)'].                     |          
+---------------------------------+--------------------------------------------------------------------------------------------+
| ../roles/Raid_R730              | Reset the RAID forcefully and re-create RAID-1 across first two disks.                     |
|                                 | convert any SSDs into Non-RAID. Runs only when `raid_force` is set to true in              |
|                                 | `inventory\hosts`. [when: '(raid_force | bool) and (model is defined and model == 730)'].  | 
+---------------------------------+--------------------------------------------------------------------------------------------+
| ../roles/iDrac_Settings         | Modify/Update the iDrac settings using the variable values mentioned in invenroty/hosts.   |
+---------------------------------+--------------------------------------------------------------------------------------------+
| ../roles/iDrac_BIOS_Settings    | Modify/Update the BIOS settings using the variable values mentioned in invenroty/hosts.    |
+---------------------------------+--------------------------------------------------------------------------------------------+
```

### Per Rack basis - Rack having 620s and 730s

Explain what these tests test and why

```
Give an example
```

### Per Rack basis - either R620s or R730s.

Explain what these tests test and why

```
Give an example
```

## Per Role Basis - Run only specific roles

Explain what these tests test and why

```
Give an example
```

## Built With

* [Dropwizard](http://www.dropwizard.io/1.0.2/docs/) - The web framework used
* [Maven](https://maven.apache.org/) - Dependency Management
* [ROME](https://rometools.github.io/rome/) - Used to generate RSS Feeds

## Contributing

Please read [CONTRIBUTING.md](https://gist.github.com/PurpleBooth/b24679402957c63ec426) for details on our code of conduct, and the process for submitting pull requests to us.

## Versioning

We use [SemVer](http://semver.org/) for versioning. For the versions available, see the [tags on this repository](https://github.com/your/project/tags). 

## Authors

* **Billie Thompson** - *Initial work* - [PurpleBooth](https://github.com/PurpleBooth)

See also the list of [contributors](https://github.com/your/project/contributors) who participated in this project.

## License

This project is licensed under the MIT License - see the [LICENSE.md](LICENSE.md) file for details

## Acknowledgments

* Hat tip to anyone who's code was used
* Inspiration
* etc


