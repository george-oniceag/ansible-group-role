#Ansible "GROUP-ROLE"
===================

A role for [Ansible](http://www.ansible.com) to configure Operating System groups for linux based OS's and applying the needed [sudo](https://www.sudo.ws/man/1.8.18/sudo.man.html) rights to groups.

## Compatibility

> List of campatible OS's
>   - CentOS/RedHat 7

## Usage 

The role includes a default variable file located in defaults/main.yml wich defines the following example:
```yaml
system_groups:
        - name: masteradmins
          status: present
          group_id: 1101
          system_group: no
          sudo:
              - place: ALL
                run_as: 'ALL:ALL'
                command: ALL
                nopasswd: "yes"
        - name: powerusers
          status: present
          group_id: 1102
        - name: guests
          status: present
          sudo:
                - place:
                  run_as:
                  command:
                - place: '127.0.0.1'
                  run_as: 'andy_the_nobody:nobody'
                  comand: ['/usr/sbin/arping','/usr/sbin/ausearch']
                  nopasswd: "no"
        - name: guests_extended
          sudo:
                - place: GROUP_LOCALHOST
                  run_as: 'guest:guest'
                  comand: ['SHELLS','REBOOT']
        - name: ssh_admins
        - name: ssh_users
        - name: junk_group
          status: absent

sudo_cmnd_alias:
        SHELLS: ['/sbin/sh','/usr/bin/sh','/usr/bin/csh','/usr/bin/ksh','/usr/local/bin/tcsh','/usr/bin/rsh','/usr/local/bin/zsh']
        REBOOT: ['/usr/sbin/reboot']
        SOFTWARE: ['/bin/rpm','/usr/bin/up2date','/usr/bin/yum']
        SERVICES: ['/sbin/service','/sbin/chkconfig','/usr/bin/systemctl start','/usr/bin/systemctl stop','/usr/bin/systemctl reload','/usr/bin/systemctl restart','/usr/bin/systemctl status','/usr/bin/systemctl enable','/usr/bin/systemctl disable']

sudo_host_alias:
        GROUP_LOCALHOST: ['localhost','127.0.0.1']
        GROUP_LOCALNET: ['192.168.0.0/24']

sudo_group_alias:
        GROUP_MASTERADMINS: ['root','masteradmins']
        GROUP_USERS: ['users']
        GROUP_NOBODY:

```

#### Variable definitions - Explained 
The **system_groups** variable is used to define the groups on the local or remote OS. You can use this variable to add new groups, update existing oanes or remove existing groups. Also with this variable you can add [sudo](https://www.sudo.ws/man/1.8.18/sudo.man.html) rights to the groups by specifing the *place,run_as,command* and *nopasswd* parameters. Information regarding these parameteres and all the options can be found bellow. 

 Variable format for system_groups:
```yaml
 system_groups:
       - name: <<name_of_the_group>>
         status: <<status_of_the_group|default('present')>> ## Define the status of the group
                       ## options for "status_of_the_group" -   <<active|present|yes|true>>
                       ##                                       <<absent|removed|no|false|inactive|deleted>>
         group_id: <<the_static_id_of_the_group>> ## WARNING !! Please check group id usage before using this option
         system_group: <<yes/no | default('no')>> ## Define wheather the group will be a system group
         sudo :         ## Define sudo rights if needed
             - place: <<location_of_run>>  ## The location from witch the user can run command
               run_as: <<user|user:group>> ## Run the command as user or as user from group
               command: <<CMND_ALIAS|command>>|['<<CMND_ALIAS|command>>',<<CMND_ALIAS|command>>',...]
               nopasswd: <<'yes/no'>> ## Set no password required flagh
             - place: .... etc
```

The **sudo_cmnd_alias** variable is used to define the command aliases that can beused in the definition of the [sudo](https://www.sudo.ws/man/1.8.18/sudo.man.html) rights. The aliasses defined here can be used in the **_system_groups.sudo_** variable to enable a more complex definition of rights for the groups.
Variable format:
```yaml
sudo_cmnd_alias:
        <<GROUP_ALIAS_NAME|upper>> : <<command>>|['<<command>>',<<command>>',...] 
        ## Each group_alias can have one or more commands listed 
```
**WARNING !!!** The _GROUP_ALIAS_NAME_ need to be in uppercase and no other simbole is eaccepted except ("_"). If non uppercase characters are used, the function will convert them automatically to uppercase.

The **sudo_host_alias** variable is used to define a set of multiple hosts, thus enableing a more complex definition of [sudo](https://www.sudo.ws/man/1.8.18/sudo.man.html) rights. The aliasses defined here can be use in the **_system.sudo variable_** to define the location from whitch the sudo commands are permitted. 
Tha format of this variable is similar with the format of **sudo_cmnd_alias** !!!

The **sudo_group_alias** is a defined but unused varible for the current moment. 

**ATTENTION** All of these aliasses cannot be in duplicat across the sudo files. Therfore the role will automatically check and analize the **_/etc/sudoers_** file and comment or remove any duplicate **alias**. All of the new aliasses will be found in the **_/etc/sudoers.d/group_sudoers_** file
 
