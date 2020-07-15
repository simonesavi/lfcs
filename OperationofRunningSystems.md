# Operation of Running Systems

## Boot, reboot, and shut down a system safely

* `shutdown -h now` shutdown
* `shutdown -r now` reboot

## Boot or change system into different operating modes

Boot sequence:

* POST (PowerOn Self Test) -> Find disk -> Inside disk there's bootloader -> bootloader load kernel -> kernel load init process

* Systemd is the default init process in CentOS
* Systemd starts services. Last service started will be a shell



Systemd

* Previous versions of Red Hat Enterprise Linux, which were distributed with SysV init or Upstart, implemented a predefined set of runlevels that represented specific modes of operation. These runlevels were numbered from 0 to 6 and were defined by a selection of system services to be run when a particular runlevel was enabled by the system administrator. In CentOS and Red Hat Enterprise Linux 7, the concept of runlevels has been replaced with systemd targets.

* Systemd targets are represented by target units. Target units end with the .target file extension and their only purpose is to group together other systemd units through a chain of dependencies. 

* Systemd units are the objects that systemd knows how to manage. These are basically a standardized representation of system resources that can be managed by the suite of daemons and manipulated by the provided utilities.

* Systemd units in some ways can be said to similar to services or jobs in other init systems. However, a unit has a much broader definition, as these can be used to abstract services, network resources, devices, filesystem mounts, and isolated resource pools.

* Systemd was designed to allow for better handling of dependencies and have the ability to handle more work in parallel at system startup.



Systemd commands:

* `systemctl get-default`

  It shows default target

* `systemctl list-units --type target --all`

  It shows all available targets

* `systemctl set-default multi-user.target`

  Set multi-user target as default



Change target at boot time

* If during boot ESC is pressed the grub2 prompt will be showed

* Highlight a voice and press 'e'

* Now is it possible to modify the boot parameter used to load the kernel. 

  **NOTE**: the changes are not persistent

  E.g `systemd.unit=emergency.target` can be added to boot system in emergency mode. NOTE: in this modality disk is mounted read only, to mount it read/write, after boot execute `mount`
  `-o remount,rw /`

* When the parameter change is end, press 'Ctrl + x' to boot system



References:

* [https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/system_administrators_guide/sect-managing_services_with_systemd-targets](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/system_administrators_guide/sect-managing_services_with_systemd-targets)
* [https://en.wikipedia.org/wiki/Power-on_self-test](https://en.wikipedia.org/wiki/Power-on_self-test)
* [https://www.digitalocean.com/community/tutorials/understanding-systemd-units-and-unit-files](https://www.digitalocean.com/community/tutorials/understanding-systemd-units-and-unit-files))

## Install, configure and troubleshoot bootloaders

* The default bootloader is Grub2.

* The to change bootloader configuration edit /etc/default/grub

  `vi /etc/default/grub`

* The configuration information can be found with:

  * `info -f grub -n 'Simple configuration'`

  * `man 7 bootparam` 

    It shows the kernel boot parameter

* When the changes were inserted in /etc/default/grub they must be inserted in configuration file used directly by Grub2 that is /boot/grub2/grub.cfg. To to this execute:

  `grub2-mkconfig -o /boot/grub2/grub.cfg`

* 

## Diagnose and manage processes

mpstat

* `yum -y install sysstat`

* `mpstat -P ALL -u 2 3`

  CPU usage statistics. 

  `-P` Indicate the processor number for which statistics are to be reported, ALL for all cpu

  `-u` Report CPU utilization

  `2 3` Display three reports at two second intervals.



ps

* `ps` Processes of which I'm owner

* `ps aux` All processes.

  It will print:

  * user - user owning the process

  * pid - process ID of the process
    * It is set when process start, this means that provide info on starting order of processes

  * %cpu - It is the CPU time used divided by the time the process has been running.

  * %mem - ratio of the process’s resident set size to the physical memory on the machine

  * VSZ (virtual memory) - virtual memory usage of entire process (in KiB)

  * RSS (resident memory) - resident set size, the non-swapped physical memory that a task has used (in KiB)

  * tty - On which process is running. 
    * **NOTE**: *?* means that isn't connect to a tty

  * stat - process state

  * start- starting time or date of the process

  * time - cumulative CPU time

  * command - command with all its arguments

    * Those within *[ ]* are system processes or kernel thread

* `ps -eo pid,ppid,cmd,%cpu,%mem --sort=-%cpu`

   `-e` show same result of `aux`

   `-o` chose columns to show

  `--sort` sort by provided parameter

  `ppid` parent process id

* `ps -e -o pid,args --forest`

  `--forest` show a graphical view of processes tree



* In /proc/[pid]

  There is a numerical subdirectory for each running process; the subdirectory is named by the process ID.

* /proc/[pid]/fd

  This is a subdirectory containing one entry for each file which the process has open, named by its file descriptor, and which is a symbolic link to the actual file.  Thus, 0 is standard input, 1 standard output, 2 standard error, and so on.



* `lsof -p pid`

  Lists open files associated with process id of pid



Background processes

* End a command with `&` execute a process in background

  `sleep 600 &`

* `jobs`

  List processes in background

* `fg pid` 

  To return a process in foreground 



Process priority

* `ps -e -o pid,nice,command`

  nice (NI) is the process priority

* More priorities and more CPU time will be assigned to process

* nice value can be between -20 and 90

* -20 is highest and 90 is lowest

* **NOTE**: only root can assign negative values

* `nice -n value command &`

  It will execute command in background with nice equal to value

* `renice` ri-assign priority to a process

  `renice -n value pid`



Signals

* `kill pid`

  Send a SIGTERM to process with pid equal to pid

* `kill -9 pid`

  Send a SIGKILL to process with pid equal to pid

* `kill -number pid`

  Send a signal that correspond to number to process with pid equal to pid

* `kill -l`

  List all available signal and corresponding number




References:

* [https://superuser.com/questions/117913/ps-aux-output-meaning](https://superuser.com/questions/117913/ps-aux-output-meaning)
* [http://man7.org/linux/man-pages/man5/proc.5.html](http://man7.org/linux/man-pages/man5/proc.5.html)

## Locate and analyze system log files

* Usually log files are stored in `/var/log`

* In Centos many tools use `rsyslog` to manage logs. 

* `rsyslog` is a daemon that permit the logging of data from different types of systems in a central repository
  * `/etc/rsyslog.conf` configuration file of rsyslog
  * `systemctl status rsyslog` to check execution status of rsyslog



References:

* [https://www.ittsystems.com/what-is-syslog/](https://www.ittsystems.com/what-is-syslog/)



## Schedule tasks to run at a set date and time

* Daemon that schedule tasks, called jobs, to run at a set date and time is cron
* The schedule of various tasks depend by configuration contained in below files/directories:
  * /etc/crontab
    * Normally isn't edited
      * **NOTE**: It's content can be used as remainder of cron files syntax
    * Each row is a task that must be executed in a scheduled way
    * A special syntax indicates the schedule of each commands
  * /etc/cron.d
    * It contains files with same syntax of /etc/crontab
    * Normally it used by software packages installed in system
  * /var/spool/cron
    * It contains tasks for users
    * Contents can be edited using `crontab` command
  * /etc/cron.hourly
    * Each script in this directory will be executed every hour
    * Exact time isn't specified but execution is granted, with a combination of deamon cron and anacron
  * /etc/cron.daily
    * Each script in this directory will be executed every day
    * Exact time isn't specified but execution is granted, with a combination of deamon cron and anacron
  * /ect/cron.weekly
    * Each script in this directory will be executed every week
    * Exact time isn't specified but execution is granted, with a combination of deamon cron and anacron
  * /etc/cron.monthly
    * Each script in this directory will be executed every month
    * Exact time isn't specified but execution is granted, with a combination of deamon cron and anacron



To modify cron jobs:

* `crontab -e` It is used by user to modify his jobs
* `crontab -e -u user` It is used by root to modify user's jobs

* Both commands will create a file in /var/spool/cron
* `crontab -u user -l` print user's jobs or better show content of file in /var/spool/cron



Cron syntax:

```bash
# ┌───────────── minute (0 - 59)
# │ ┌───────────── hour (0 - 23)
# │ │ ┌───────────── day of the month (1 - 31)
# │ │ │ ┌───────────── month (1 - 12)
# │ │ │ │ ┌───────────── day of the week (0 - 6) (Sunday to Saturday;
# │ │ │ │ │                                   7 is also Sunday on some systems)
# │ │ │ │ │
# │ │ │ │ │
# * * * * * command to execute
```

* `#` this line is a comment
* `*` always
* `1 0 * * * /command` command will be executed one minute past midnight (00:01) every day
*  `1-30 * * * * /command` command will be executed every day, every hour at minutes 1 to 30
*  `*/10 * * * * /command` command will be executed every 10 minutes, or rather when minutes are 00, 10, 20, 30, 40 and 50.
*  `00 */2 15 * * /command` command will be executed the fifteenth day of every month, every two hours
*  `00 1-9/2 1 5 * /command` command will be executed on 1st May at 1,00 - 3,00 - 5,00 - 7,00 - 9,00, or rather every two hours from 1,00 to 9,00
*  `00 13 2,8,14 * * /command` command will be executed second, eighth and fourteenth day of each month at 13.00



at

* `yum -y install at`
* **NOTE**: it require that atd demon will be in execution
  * `systemctl start atd`
  * `systemctl enable atd`
* `at 11:00` open a shell in which inserted commands that will be executed at 11:00
  * `ctrl+d` close shell

* `atq` shows scheduled activities identified by an activity ID
* `atrm ID` will remove from schedule activity with activity ID equal to ID



References:

* [https://en.wikipedia.org/wiki/Cron](https://en.wikipedia.org/wiki/Cron)

* [http://guide.debianizzati.org/index.php/Utilizzo_del_servizio_di_scheduling_Cron](http://guide.debianizzati.org/index.php/Utilizzo_del_servizio_di_scheduling_Cron) (Italian language)



## Verify completion of scheduled jobs

* Cron will send an email to internal mail spool 



* Enable the logging  of crond events
* Edit the /etc/rsyslog.conf and remove comment from this line:

```bash
# Log cron stuff
cron.*                                                  /var/log/cron
```

* `systemctl restart rsyslog`  it will restart rsyslog server

## Update software to provide required functionality and security

* `yum update` 
* Yum also offers the upgrade command that is equal to update with enabled `obsoletes` configuration option. By default, obsoletes is turned on in `/etc/yum.conf`, which makes these two commands equivalent.
* The `obsoletes` option enables the obsoletes process logic during updates.When one package declares in its spec file that it *obsoletes* another package, the latter package is replaced by the former package when the former package is installed. Obsoletes are declared, for example, when a package is renamed



References:

* [https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/system_administrators_guide/ch-yum](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/system_administrators_guide/ch-yum)

* [https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/system_administrators_guide/sec-Configuring_Yum_and_Yum_Repositories#sec-Setting_main_Options](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/system_administrators_guide/sec-Configuring_Yum_and_Yum_Repositories#sec-Setting_main_Options)



## Verify the integrity and availability of resources

* `/usr/lib/rpm/rpmdb_verify /var/lib/rpm/Packages` It will verify the integrity of rpm database

## Verify the integrity and availability of key processes

* `systemctl status processname` It will show the status of process with name processname
  * The las rows are the recent logs generated by daemon



* Other command to check processes status:
  * `ps`
  * `pgrep`
  * `mpstat`

## Change kernel runtime parameters, persistent and non-persistent

* In /proc/sys are contained kernel tunables, parameters that are used to customize the behavior of system

* Example

  * `cd /proc/sys/net/ipv6/conf/all`

  * `echo 1 > /proc/sys/net/ipv6/conf/alldisable_ipv6` 

    Will disable IPv6

  * **NOTE**: This is a runtime change, not permanent

  * **NOTE**: With this files `vi` cannot be used

* Alternative method: `sysctl -w net.ipv6.conf.all.disable_ipv6=1`

* `sysctl -a` shows all parameters that can be configured



To make configuration permanent

* `cd /etc/sysctl.d`
* `echo net.ipv6.conf.all.disable_ipv6 = 1 > ipv6.conf`
  * **NOTE**: the only request is that file will end with `.conf`
* `sysctl -p` reload permanent configuration. Alternative: reboot system



Some parameters changed commonly:

* net.ipv4.ip_forward=0 disable packet forwarding

* fs.file-max -> massimo numero di file gestibili

* kernel.sysrq -> abilita printscreen key

* net.ipv4.icmp_echo_ignore_all -> ignora ping

## Use scripting to automate system maintenance tasks

Bash shell script:

* `#!/bin/bash` must be first row
* A Bash script is a plain text file which contains a series of commands or/and typical constructs of imperative programming
* It is convention to give files that are Bash scripts an extension of **.sh**

*  `chmod +x nomefile.sh` must be executable
* `./nomefile.sh` execute nomefile.sh



References:

* [https://ryanstutorials.net/bash-scripting-tutorial/bash-script.php](https://ryanstutorials.net/bash-scripting-tutorial/bash-script.php)

## Manage the startup process and services (In Services Configuration)

* `systemctl` command used to manage servers. In Linux servers often are called *daemons*

* `systemctl status processname` It will show the status of process with name processname
  * `Active` process status eg. inactive, active
  * `Loaded` unit file name
    * unit file name; enable - This means that daemon will be executed automatically at the next reboot
    * unit file name; disabled  This means that daemon won't be executed automatically at the next reboot
  * The las rows are the recent logs generated by daemon
* `systemctl start sshd` It will start sshd daemon
* `systemctl stop sshd` It will stop sshd daemon
* `systemctl restart sshd` It will restart sshd daemon
  * **NOTE**: A restart must be executed each time a daemon configuration file is changed
* `systemctl disable sshd` Disable the execution of service at bootstrap
* `systemctl enable sshd` Enable the execution of service at bootstrap
* `systemctl is-enabled sshd` Check if daemon is enable or disabled in bootstrap sequence
* `systemctl list-unit-files` List all systemd units object available



References:

* [https://www.digitalocean.com/community/tutorials/how-to-use-systemctl-to-manage-systemd-services-and-units](https://www.digitalocean.com/community/tutorials/how-to-use-systemctl-to-manage-systemd-services-and-units)

## List and identify SELinux/AppArmor file and process contexts

* In computer security, mandatory access control (MAC) refers to a type of access control by which the operating system constrains the ability of a subject or initiator to access or generally perform some sort of operation on an object or target. In practice, a subject is usually a process or thread; objects are constructs such as files, directories, TCP/UDP ports, shared memory segments, IO devices, etc. Subjects and objects each have a set of security attributes. Whenever a subject attempts to access an object, an authorization rule enforced by the operating system kernel examines these security attributes and decides whether the access can take place. Any operation by any subject on any object is tested against the set of authorization rules (aka policy) to determine if the operation is allowed.
* In CentOS as MAC is used SELinux
* SELinux can be in three states:
  * *enforcing*: Actions contrary to the policy are blocked and a corresponding event is logged in the audit log
  * *permissive*: Actions contrary to the policy are only logged in the audit log
  * *disabled*: The SELinux is disabled entirely
  * The status can be configured in file `/etc/sysconfig/selinux`. Changing to this file will be read only after reboot
  * When state is set to *enforcing* can be switched to *permissive* and vice versa without reboot system
  * When the state is set to disable the only way to re-enable SELinux is to change `/etc/sysconfig/selinux` and reboot
* `getenforce` show the SELinux state
* `setenforce Permissive` set the state to permissive
* `setenforce Enforcing` set the state to enforcing



* On systems running SELinux, all processes and files are labeled in a way that represents security-relevant information. This information is called the *SELinux context.*
* Normally SELinux context is showed with `-Z` option
* `ls -lZ` show SELinux context of file
* `ps auxZ` show SELinux context of processes
* A SELinux context has the form *user:role:type* 
* type indicate the type of object
* unconfined_t are object not limited by SELinux



* References
  * [https://en.wikipedia.org/wiki/Mandatory_access_control](https://en.wikipedia.org/wiki/Mandatory_access_control)

## Manage Software

yum

* packet manager that use RPM packet manager

* `yum search keyword`

  This  is  used  to  find  packages when you know something about the package but aren't sure of it's name. By default search will try searching just package names and summaries, but if that "fails" it will  then  try  descriptions  and url.

* *Repository*: collections of software packages used by yum. They are configured in `/etc/yum.repos.d`
* `yum info package` Information on package

  * If package is installed Repo will be equal to "installed"
* `yum install package` Install package
* `yum provides */file` Search package that contain file
* `yum remove package`Remove package
* `yum autoremove package`Remove package plus unused dependencies
* `yumdownloader package` download the RPM package

  * **NOTE**: require `yum -y install yum-utils`



RPM

* `rpm -i file.rpm` Install file.rpm
* `rpm -U file.rpm` Upgrade file.rpm
* `rpm -qa` List all installed RPM
* `rpm -qf file` Tells to what RPM package file belong

## Identify the component of a Linux distribution that a file belongs t`o`

* `yum provides */file` Search package that contain file



* `ldd path/command` Show all libraries used by command
* This info is contained in a library cache
* The library cache can be re-build using `ldconfing`
* The library cache is in /etc/ld.so.cache
* The info for cache are in /etc/ld.so.cache.d/
* The cache is normally re-build each time a new package is installed

