# User and Group Management

## Create, delete, and modify local user accounts

useradd

* Add users

*  `useradd -D` print the default configuration used by useradd command

  ```bash
  GROUP=100
  HOME=/home
  INACTIVE=-1
  EXPIRE=
  SHELL=/bin/bash
  SKEL=/etc/skel
  CREATE_MAIL_SPOOL=yes
  ```

  *GROUP=100* -> default group

  *HOME=/home* -> base for home directory

  *INACTIVE=-1* -> user password won't expire

  *EXPIRE=* -> user account won't expire

  *SHELL=/bin/bash* -> default shell

  *SKEL=/etc/skel* -> skeleton directory. <u>It's content will be copied in new user home directory</u>

  *CREATE_MAIL_SPOOL=yes* -> User will have a mail spool to receive email

* This configuration is saved in  `/etc/default/useradd`

* Also  `/etc/login.defs` parameter are evaluated during user add

* Some parameter of `/etc/login.defs` will overwrite `/etc/default/useradd` parameters

* `/etc/login.defs` contains:

  * Location of mail spool
  * Settings about password
  * *CREATE_HOME   yes* -> create home directory
  * *USERGROUPS_ENAB  yes* -> means that a group with same name of user must be created. This group will become default user group. This means that value of GROUP in `/etc/default/useradd` is overwritten

* `useradd` parameters:

  * `-c` Any text string. It is generally a short description of the login, and is currently used as the field for the user's full name.
  * `-e` date after which the/ user will be disabled
  * `-g` primary group. NOTE: if not specified it will be created a new group with same name of user that will be become user's primary group
  * `-G` secondary groups
  * `-m` create home directory. Useless because CREATE_HOME is yes
  * `-p` configure password. **NOTE**: value must be provided encrypted
    * Normally password is not provided during user add
  * `-s` shell to use

* When a user is created two file will be changed:

  * `/etc/passwd` It contains users information, no passwords
    * Syntax:  
      * user name
      * x: means that password isn't stored here
      * userid: user id (UID) 
      * groupid: primary group id (GID)
      * User Info: The comment field
      * home: home directory
      * shell: shell
    * To edit file: `vipw`
  * `/etc/shadow` It contains passwords plus passwords properties
    * To edit file: `vipw -s`



usermod

* used to modify a user
* `usermod` parameters:
  * `-L`lock user password
  * `-U` unlock user password
* `usermod -e 1 user` disable user
* `usermod -e "" user` enable user



userdel

* remove user
* `userdel -r user`
  * `-r` remove home and email spool. **NOTE**: if it won't be used, if it will be tried to insert same user, there will be a conflict
  * `-f` force. Delete user though he is logged



passwd

* Change password of current user
* `passwd user`
  * Used by root
  * Change password of user
* `passwd -l user`
  * Used by root
  * Lock password of user
* `echo newpass | passwd --stdin brenda`
  * It will change password of brenda
  * Can be used in a script
  * **NOTE**: Dangerous, password is in clear text



chage

* Change user password expiry information
* If used without parameters will prompt for information
* It will permit to change date when the password was last changed
* `chage -E 2014-09-11 user`
  * Set a date after which user will be locked



## Create, delete, and modify local groups and group memberships

groupadd

* add group
* When a group is created `/etc/group` file will be changed
  * Syntax:
    * group_name: It is the name of group. If you run ls -l command, you will see this name printed in the group field.
    * Password: Generally password is not used, hence it is empty/blank. It can store encrypted password. This is useful to implement privileged groups.
    * Group ID (GID): group id
      * For each user must be assigned a group ID. You can see this number in your /etc/passwd file.
    * Group List: It is a list of user names of users who are members of the group. The user names, must be separated by commas.
      * **NOTE**: The groups without group list are used as primary group for some users



groupdel

* delete group



groupmod

* modify group



* `usermod -aG group user`
  * Add group to user
  * -G list of secondary groups
  * `-a` append. <u>**NOTE**: If not specified new group list will override current value</u>



## Manage system-wide environment profiles

* The variable for all users are stored in `/etc/environment` 

* The variable for a user are stored in his home directory in file `.bash_profile`
  * **NOTE**: It is an hidden file, it is visible only running `ls -la`

## Manage template user environment

* `/etc/skel` skeleton directory. <u>It's content will be copied in new user home directory</u>

## Configure user resource limits

ulimit

* It limits the use of system-wide resources

* Limits can be configured changing file `/etc/security/limits.conf`

* Typical configuration

  ```bash
  1. @student        hard    nproc           20
  2. @faculty        soft    nproc           20
  3. ftp             hard    nproc           0
  4. @student        -       maxlogins       4
  
  ```

  1. Members of student group can run only 20 processes
  2. Members of faculty group will receive and info after that more than 20 processes were run (soft limit)
  3. ftp user cannot run any process
  4. Members of student can have maximum 4 logged user. - means both hard and soft

* `man limits.conf` for manual

* Limits will be enforced in next opened session
* Also `ulimit` command can be used to change limits

## Manage user privileges

Refer to `sudo` configuration

## Configure PAM

* PAM = plugable authentication modules
* A command/program can be PAM aware
* PAM can be used to configure e.g. login to use Active Directory or LDAP
* Use ldd to see if command use PAM libraries
  * `ldd /usr/bin/passwd | grep pam`

* Each command that will use PAM will have an entry in `/etc/pam.d` with its PAM configuration
* A good example of PAM configuration is showed in pam_tally2 module man page
  * pam_tally2: The login counter (tallying) module
  * At the end of man page there is an example to configure login to lock the account after 4 failed logins
  * `man pam_tally2`
