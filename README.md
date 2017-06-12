#Ansible "ANSIBLE-GROUP-USER-ROLE"
===================

A role for [Ansible](http://www.ansible.com) to configure Operating System Groups & Users for linux based OS's and applying the needed [sudo](https://www.sudo.ws/man/1.8.18/sudo.man.html) rights.
This role can provide the following features:
- configure Groups;
- configure sudo rights for Groups
- configure Users with: custome [Home](https://en.wikipedia.org/wiki/Home_directory), custom [Shell](https://en.wikipedia.org/wiki/Unix_shell), master Group and optional Groups 
- configure System type Users
- configure [sudo](https://www.sudo.ws/man/1.8.18/sudo.man.html) rights for Users
- move User Home
- User password management (Please use this with [ansible-vault](http://docs.ansible.com/ansible/playbooks_vault.html) )
- User SSH public key management
- advanced [sudo](https://www.sudo.ws/man/1.8.18/sudo.man.html) configuration available: configurable Host, Command, User and Group Aliasss
- advanced sanity checks for sudo configuration in order to warn user regarding the location of the error

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
                options: ['NOPASSWD','SETENV']
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
                  command: ['/usr/sbin/arping','/usr/sbin/ausearch']
        - name: guests_extended
          sudo:
                - place: GROUP_LOCALHOST
                  run_as: 'guest:guest'
                  command: ['SHELLS','REBOOT']
        - name: ssh_admins
        - name: ssh_users
        - name: junk_group
          status: absent

system_users:
        - name: masteradmin_user
          status: present
          group: masteradmins
          comment: "Master admini user"
          createhome: 'yes'
          home: '/home/masteradmin'
          system_user: 'no'
          user_id: 1101
          groups: ['root','masteradmins']
          shell: '/bin/bash'
          password: '{{vault_user_passwd.masteradmin_user}}'
          update_password: "always"
          sudo:
              - place: ALL
                run_as: 'ALL:ALL'
                command: ALL
                options: ['NOPASSWD','SETENV']
        - name: poweruser
          status: present
          group: powerusers
          shell: '/bin/sh'
          password: '{{vault_user_passwd.poweruser}}'
          update_password: 'on_create'
        - name: guests
          status: present
          groups:
          password: 'guest'
          sudo:
                - place:
                  run_as:
                  command:
                - place: '10.0.0.1'
                  run_as: 'nobody:nobody'
                  command: ['/usr/sbin/arping','/usr/sbin/ausearch','/usr/bin/ping']
        - name: testsshuser
        - name: junk_user
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
               options: ['option1','option2'...] ## Set sudo options
             - place: .... etc
```

 Variable format for system_users:
```yaml
 system_users:
       - name: <<name_of_the_user>>
         status: <<status_of_the_user|default('present')>> # it not defined - value is 'present'
                       # options for "status_of_the_user" : <<active|present|yes|true>>
                       #                                    <<absent|removed|no|false|inactive|deleted>>
         group: <<primary_user_group|default('users')>> # Specify the primary user group
         comment: <<shot_information_about_the_user|default('Ansible defined USER')>> # Short user descriptions or comment
         createhome: <<yes/no | default('yes')>> # Specify wheather to create user home or not. Default 'yes'
         home: <<the_location_of_the_home_directory|default(omit)>> # Specify the location of the users home directory.
                                                                    # DEFAULT - use the system default location
         system_user: <<set_as_system_user|default('no')>> # Define the user as a system user
                       # WARNING !! Changing this setting after user is created will not be possible
         user_id: <<the_static_id_of_the_user>> # Provide a manual Group ID for the coresponding user
                                                # WARNING !! Please check group id usage before using this option
         groups: <<list_of_additional_groups>> # Provide additional groups for the user to be added in
                     # WARNING !! If left Empty the user will be removed from all groups except the master group
         shell: <<the_shell_wich_the_user_can_user>> # The default shell on which the user will function
                # WARNING !! Please provide an existing shell or user will not be functionall
         password: <<password_string>>               # WARNING !! Please use with ansible-vault for seacurity reasons
         update_password: <<always|on_create>> # Specify wheatehr the password will be updated or not. Please use the specified options
         sudo :  ## define sudo rights if needed
             - place: <<location_of_run>>  ## The location from witch the user can run command
               run_as: <<user|group|user:group>> ## Run the command as user or as user from group
               command: <<CMND_ALIAS|command>>|['<<CMND_ALIAS|command>>',<<CMND_ALIAS|command>>',...]
               options: ['option1','option2'...] ## Set sudo options
             - place: .... etc
```

The **sudo_cmnd_alias** variable is used to define the command aliases that can beused in the definition of the [sudo](https://www.sudo.ws/man/1.8.18/sudo.man.html) rights. The aliasses defined here can be used in the **_system_groups.sudo_** variable to enable a more complex definition of rights for the groups.
Variable format:
```yaml
sudo_cmnd_alias:
        <<GROUP_ALIAS_NAME|upper>> : <<command>>|['<<command>>',<<command>>',...] 
        ## Each group_alias can have one or more commands listed 
```
**WARNING !!!** The _GROUP_ALIAS_NAME_ need to be in uppercase and no other simbole is accepted except ("_"). If non uppercase characters are used, the function will convert them automatically to uppercase.

The **sudo_host_alias** variable is used to define a set of multiple hosts, thus enableing a more complex definition of [sudo](https://www.sudo.ws/man/1.8.18/sudo.man.html) rights. The aliasses defined here can be use in the **_system.sudo variable_** to define the location from whitch the sudo commands are permitted. 
Tha format of this variable is similar with the format of **sudo_cmnd_alias** !!!

The **sudo_group_alias** is a defined but unused varible for the current moment. 

**ATTENTION** All of these aliasses cannot be in duplicat across the sudo files. Therfore the role will automatically check and analize the **_/etc/sudoers_** file and comment or remove any duplicate **alias**. All of the new aliasses will be found in the **_/etc/sudoers.d/group_sudoers_** file
 
