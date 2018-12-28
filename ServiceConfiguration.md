# Service Configuration

## Configure a caching DNS server

![DNS Resolution](Pictures/dns_resolution.gif)

* Linux DNS server is *bind*

  * `yum -y install bind bind-utils`

* Main configuration file `/etc/named.conf`

* Most important configurations:

  ```bash
  options {
          listen-on port 53 { 127.0.0.1; 192.168.0.0/24; };
  		...
          allow-query        { localhost; 192.168.0.0/24; };
          allow-query-cache  { localhost; 192.168.0.0/24; };
  		...
          recursion yes;
          forwarders {
                  8.8.8.8;
                  8.8.4.4;
          };
  		...
  };
  
  zone "test.com." IN {
          type master;
          file "/var/named/test.com.zone";
  };
  
  zone "0.168.192.in-addr.arpa" IN {
          type master;
          file "/var/named/rev.test.com.zone";
  };
  ```

  * `listen-on port 53` tell on which network interfaces and port to accept client queries.

  * `allow-query` defines the networks from which clients can post DNS requests.
  
  * `allow-query-cache` defines the addresses/networks from which clients are allowed to issue queries that access the local cache.

  * `forwarders` specifies the name servers to which DNS requests should be forwarded if they cannot be resolved directly. 

  * `zone` contains domain configuration. After `zone`, specify the name of the domain to administer.

    * `file` specifies the file where zone data for the domain is located.

  * `zone "0.168.192.in-addr.arpa"` is the configuration for reverse zone or reverse lookup. A reverse zone allows DNS to convert from an address to a name.

    * `0.168.192` must be substituted with the first three octets of whatever network addresses range are managed


* `systemctl start named` start bind server




References:

* [http://web.deu.edu.tr/doc/oreily/networking/dnsbind/ch02_06.htm](http://web.deu.edu.tr/doc/oreily/networking/dnsbind/ch02_06.htm)
* [https://www.pks.mpg.de/~mueller/docs/suse10.1/suselinux-manual_en/manual/sec.dns.named.html](https://www.pks.mpg.de/~mueller/docs/suse10.1/suselinux-manual_en/manual/sec.dns.named.html)



## Maintain a DNS zone

- `/var/named/test.com.zone` contents

  ```bash
  $TTL 3H
  @       IN SOA  dns root.test.com. (
                                          0       ; serial
                                          1D      ; refresh
                                          1H      ; retry
                                          1W      ; expire
                                          3H )    ; minimum
          IN NS   dns
          IN MX   10 email
  
  dns     IN A    192.168.0.29
  email   IN A    192.168.0.29
  web     IN A    192.168.0.29
  www.web IN CNAME web
  ```

  - Line 2: This is where the SOA (start of authority) control record begins.
    - `@` means that zone name will be extracted from the corresponding entry in `/etc/named.conf` (in this example test.com.)
    - `dns` is the name of authoritative server for the zone
    - `root.test.com.` an e-mail address of the person in charge of this name server. Because the `@` sign already has a special meaning, `.` is entered here instead. For `root@test.com` the entry must read`root.test.com.`
  - Line 8: The `IN NS` specifies the name server responsible for this domain (authoritative server)
  - Line 9: The `MX` record specifies the mail server that accepts, processes, and forwards e-mails for this domain
  - Last lines: These are the actual address records where one or more IP addresses are assigned to hostnames.
    - CNAMES maps a name on another name

* `/var/named/rev.test.com.zone` contents:

  ```bash
  $TTL 3H
  @       IN SOA  dns.test.com. root.test.com. (
                                          0       ; serial
                                          1D      ; refresh
                                          1H      ; retry
                                          1W      ; expire
                                          3H )    ; minimum
          IN NS   dns.test.com.
  
  29      IN PTR  dns.test.com.
  ```

  * Line 2: The configuration file should activate reverse lookup for the network `192.168.1.0`. Given that the zone is called `1.168.192.in-addr.arpa`, should not be added to the hostnames. Therefore, all hostnames are entered in their complete form—with their domain and with a `.` at the end. The remaining entries correspond to those described for the `test.com.` zone
  * Line 8: This line specifies the name server responsible for this zone. This time, however, the name is entered in its complete form with the domain and a `.` at the end.
  * Line 10: This is the pointer record hinting at the IP addresses on the respective hosts. Only the last part of the IP address is entered at the beginning of the line, without the `.` at the end.



* **NOTE**:  Examples of configuration files are contained in `/usr/share/doc/bind-9.9.4/sample`

  * `bind` directory name depends by installed version


* To check name resolution is possible to use `host`
  * `host name_to_resolve dns_server_ip`
  * E.g. `host dns localhost`
  * E.g of reverse zone `host 192.168.0.29 localhost`

References:

* [https://www.pks.mpg.de/~mueller/docs/suse10.1/suselinux-manual_en/manual/sec.dns.zonefile.html](https://www.pks.mpg.de/~mueller/docs/suse10.1/suselinux-manual_en/manual/sec.dns.zonefile.html)

## Configure email aliases

* To manage mail spool

  * `yum -y install mailx`
  * `mailx` reads the user's mail spool

* Send an email to spool

  * `echo "Test" | mail -s "Oggetto" root`

    *root* is target user



* To create an alias edit file `/etc/aliases`

  * Add line like `root: user,root`

    This create an alias for `root` and this means that email for root will be sent to `user` and `root` mail spool

  * `root: user@test.com` 

    Whit this syntax will be added a classical email address

* At the end of changes to `/etc/aliases` execute `newaliases` to apply changes

## Configure SSH servers and clients

* `/etc/ssh/sshd_config` ssh server configuration file
  * `PermitRootLogin no` Disable `root` login with ssh client
  * `PasswordAuthenticaion no` Disable login with password. This means that only login with public and private keys is allowed
* `/etc/ssh/ssh_config` ssh client configuration file
  * `ForwardX11 yes` allows use of X11 Server with ssh



Server management

* `systemctl status sshd` to control ssh server status
* `systemctl stop sshd` stop ssh server
* `systemct start sshd` start ssh server
* `systemctl restart sshd` restart ssh server
  * It must be executed each time configuration file will be changed
* `systemctl disable sshd` disable the ssh server start at boot
* `systemctl enable sshd` enable the ssh server start at boot



Client commands

* `ssh 129.123.123.123 ` it try to connect current user to an ssh server located on 192.123.123.123
* `ssh root@129.123.123.123 ` it try to connect root user to an ssh server located on 192.123.123.123
* `ssh -X root@129.123.123.123 ` 
  * `-X` enable X11 forwarding. This means that graphical application can be started
  * NOTE: It must be allowed on client configuration file as well.

* First time that an ssh connection is established with a server, the server will send a public key that it is used to verify its identity.
* The server public key is stored in the user's home inside file`.ssh/know_hosts`
  * E.g. `/home/user/.ssh/know_hosts`



Authentication with public/private keys

* On the ssh client machine a couple of ssh public/private keys can be generated using `ssh-keygen`
* The keys will be stored in the user's home inside directory `.ssh`
  * `id_rsa` private key
  * `id_rsa.pub` public key
* `ssh-copy-id 123.123.123.123` it is used to copy current user public key to home directory of same user on ssh server. The key will be stored in the user's home inside file `.ssh/authorized_keys`

* After that public key is copied on the server, user can use ssh client to login into the server without providing password



scp

* Secure copy. It use ssh to copy file on a server
* `scp /test/source 123.123.123.123:/dest` It will copy local file /test/source in /dest directory on the server 123.123.123.123
* `scp 123.123.123.123:/source /dest` It will copy source file from server to local directory dest



## Restrict access to the HTTP proxy server

* To enable the use of a proxy server environment variable `http_proxy` must be configured
  * `export http_proxy=http://127.0.0.1:3128/` use a local proxy listening on port 3128
  *  `export http_proxy=http://username:password@192.168.0.1:8080/` use a remote proxy on server 192.168.0.1, listening on port 8080 that require user and password
* `unset http_proxy` Disable use of proxy

* The keep configuration permanent for all user insert variable configuration in `/etc/environment` 

## Configure an IMAP and IMAPS service

* Server used to manage IMAP protocol is dovecot

  * `yum -y install dovecot`

* Basic configuration

  * `/etc/dovecot/dovecot.conf`

    * `protocols = imap pop3`

      This will enable imap and pop3 protocol

  * `/etc/dovecot/conf.d/10-mail.conf`

    * `mail_location = maildir:~/Maildir`

      This indicate to server where is located mail file

  * `/etc/dovecot/conf.d/10-ssl.conf`

    * Nothing to change, default configuration will enable ssl version of protocols that are enable in `dovecot.conf`

## Query and modify the behavior of system services at various operating modes

* `/usr/lib/systemd/system` contain unit file *.service* used by systemctl to start various service
* `/etc/systemd/system` can contain unit file that "override" the files contained in /usr/lib/systemd/system. If a unit file for a service is present in this directory, it will be used in substitution of file present in /usr.
* The correct way to permanently alter a start property of a service is to copy original file from `/usr/lib/systemd/system` to `/etc/systemd/system` and modify copy
* From the output of `system status service` it is possible to find from which file service was start`ed`
  * `Loaded` show the name of .service file used
* Under `[install]` session, voice `WantedBy` indicates for which target service is required
* When a service is enabled, a symbolic link to file `.service` of service will be created in `/etc/systemd/system/targetname.target.wants` where *targetname*  is the name of target for which service is required



* Some service properties can be changed at runtime

  * `systemctl set-property httpd.service MemoryLimit=500M`

    Command will change property and will create a file in `/etc/systemd/system` for future boot

  * `system status service` will show

    * `Loaded` will show the name of .service file used

    * `Drop-in` will show the change in `/etc/systemd`


* `systemctl list-dependencies service` It will show service dependencies

## Configure an HTTP server

* Used server: Apache HTTP Server
* `yum -y install httpd` will install server
* `systemctl start httpd` will start server
* `/etc/httpd/conf/httpd.conf` is the principal configuration file
  * `ServerName localhost` contains the local server name. 
    * **NOTE**: it must correspond to an IP. Simple solution is to modify /etc/hosts to insert a name-IP mapping
* Virtual host can be created inserting a file *.conf* in `/etc/httpd/conf.d/`
  * E.g. `/etc/httpd/conf.d/file.conf`
* The file structure can be copied from  `/usr/share/doc/httpd-2.4.6/httpd-vhosts.conf`
  * **NOTE**: The version depends by server version installed
* Normally as *DocumentRoot*, directory that will contain site's files, it will be used a directory in `/var/www`

## Configure HTTP server log files

* E.g.

  ```bash
  ErrorLog /var/log/httpd/example.com_error_log
  LogFormat %s %v combined
  CustomLog /var/log/httpd/example.com_access_log combined
  ```

  * This will generate store Error log in /var/log/httpd/example.com_error_log

  * Plus will generate a log with a custom format in /var/log/httpd/example.com_access_log

* Normally log are stored in /var/log/httpd



* `yum -y install httpd-manual` will install httpd manuals
* Manuals are in http format
* In `/usr/share/httpd/manual/vhosts` are stored manual for vhost

## Configure a database server

* Used database: MariaDB
* `yum -y install mariadb mariadb-server` will install database
* `systemctl start mariadb` will start database
* `mysql -u root -p` will connect to database as root database user
  * Default password is blank
* `mysql_secure_installation` improves MariaDB security
  * It will permit to configure root password

## Restrict access to a web page

* Edit `/etc/httpd/conf/httpd.conf` and change

  ```bash
  <Directory "/var/www">
  	AllowOverride All
  ```

* In subdirectory of `/var/www` where site pages are contained create a file `.htaccess` whit follow content:

  ```bash
  Order Deny, Allow
  Deny from 192.168.3.1
  ```

  This will deny accesso to pages from IP 192.168.3.1 and allow access from all other IPs

* Alternatively:

  ```
  Order Allow, Deny
  Allow from 192.168.3.1
  ```

   This will allow access to pages from IP 192.168.3.1 and deny access from all other IPs

## Manage and configure containers

* Concepts:
  * *Images*: Read only template used to create container.
  * *Container*: Isolated application platform, it contains all the need to execute application



* `yum install docker` It will install docker
* `systemctl start docker`It start docker
* `docker version` to test if docker is working properly
* `usermod -aG dockerroot user`
  * This will enable *user* to use docker
* `docker search java`
  * Search java image in docker hub
* `docker images`
  * List local images
* Run container, examples:
  * `docker run busybox ls`
  * `docker run busybox echo "hello"`
  * `docker run centos:7 ping 127.0.0.1`
* `docker run -i -t centos:7 bash`
  * Run container with terminal
  * `-i` connects standard input to container
  * `-t` get pseudo terminal
  *  **NOTA**: `ctrl+p+q` exit form terminal without terminate container execution
* `docker run -d centos:7 ping 127.0.0.1`
  * Container will be executed in detached mode. This means that is in execution in background and not attached to Bash shell
* `docker ps -a`
  * List all container
  * `-a` show container stopped as well
* `docker attach containername`
  * Attach to container in detached mode
* `docker logs containername`
  * Show logs of a container
* `docker run -d  -P nginx`
  * Map container ports to host ports
  * **NOTE**: *firewalld* must be enable and running
* `docker run -d -P --restart always nginx`
  * This container will be restarted at bootstrap if the guest host will be restarted
* `docker update --restart=no containername`
  * Disable auto restart at bootstrap
* Stop container:
  * `docker stop containername`
  * `docker kill containername` forced stop
* `docker start name`
  * Restart a stopped container
* `docker rm containername`
  * Remove a container
  * **NOTE**: It must be stopped
* `docker rmi imageid`
  * Remove local image
* `docker diff containername`
  * List differences between container and original images. E.g. Some software can be installed in running container
* `docker commit containername`
  * Create a new image using based on the content of current running container. E.g It will contain software that was installed in container

## Manage and configure Virtual Machines

* `yum install qemu-kvm qemu-img libvirt virt-install libvirt-client` this will install all tools need to manage and configure virtual machines
* `systemctl start libvirtd`  this will start daemon need to manage virtual enviroments



Manage storage volume

* Concepts:

  * Storage Pool -> Container of storage volumes (e.g. directory, partitions)
  * Storage Volume -> virtual disk

* Create a Storage Pool:

  * `virsh pool-define-as spool dir - - - - "/media/vdisk/`
  * `virsh pool-build`
  * `virsh pool-start`

  * `virsh pool-autostart`

* In files `/etc/libvirt/storage/*.xml` you can find info about storage pool

* Create a virtual disk

  * `qemu-img create -f raw /media/vdisk/disk.img 1G` size will be 1G



Manage Virtual Machines

* If you what that *root* will be able to execute virtual machines, in `/etc/libvirt/qemu.conf` uncomment `user=root` and `group=root` and after restart *libvirtd* daemon with `systemctl`
  `restart libvirtd`

* Create a Virtual Machine

  * `virt-install --name=rhel7 --disk path=/mnt/personal-data/SPool1/SVol1.img,size=2 --vcpu=1 --ram=1024 --location=/run/media/dos/9e6f605a-f502-4e98-826e-e6376caea288/rhel-server-7.0-x86_64-dvd.iso --network bridge=virbr0  --graphics none --extra-args console=ttyS0`
  * This will prepare a new virtual machine named *rhel7* with 1 virtual cpu, 1G of RAM, and a virtual disk of 2G.
  * After creation, virtual machine will be booted for the first time ad a provided ISO image will be executed. Normally ISO will be an operating system installation disk
  * Virtual Machine is configured to not use graphical environment and plus a configuration to allow a connection from the local machine is set

* Virtual Machine management

  * `virsh list --all`

    List all available virtual machines in any state

  * `virsh start rhel7`

    Start a virtual machine called rhel7

  * `virsh shutdown rhel7`

    Shutdown virtual machine  called rhel7

  * `virsh destroy rhel7`

    Forced shutdown of a virtual machine called rhel7

  * `virsh undefine rhel7`

    Delete a virtual machine called rhel7

  * `virsh console rhel7`

    Establish a connection toward virtual machine called rhel7

    **NOTE**:  console must be configured in virtual machine

    `ctrl+5` to exit

  * `virsh autostart rhel7`

    Set the virtual machine to re-start if hosting machine will be rebooted

  * `virsh autostart --disable rhel7`

    Disable autostart

* Edit virtual machine

  * `virsh dominfo rhel7`

    It shows virtual machine information

  * `virsh edit rhel7`

    Edit configuration file of virtual machine called rhel7

  * `virsh vcpucount rhel7`

    It shows the number of virtual cpu

    * **maximum config**: Specifies the maximum number of virtual CPUs that can be made available for the virtual server after the next restart.

    * **maximum live**: Specifies the maximum number of virtual CPUs that can be made available for the running or paused virtual server. If you change maximum this can be different until virtual machine is rebooted

    * **current config**: Specifies the actual number of virtual CPUs which will be available for the virtual server with the next restart.

    * **current live**: Specifies the actual number of virtual CPUs which are available for the running or paused virtual server

  * `virsh setvcpus --count 2 rhel7 --maximum --config`

    It sets the maximum number of virtual cpu in configuration file to 2.

    It require virtual machine reboot to be applied. After reboot maximum live will be aligned

  * `virsh setvcpus --count 2 rhel7 --config`

    It sets the configure for virtual machine. This value its the value with which virtual machine will be booted

  * `virsh setvcpu --count 2 rhel7`

    Set the number of virtual cpu (current live).

    Number must be less or equal to maximum live.

    You cannot remove virtual CPUs from a running virtual server

  * `virsh setmaxmem --size 2G rhel7`

    It sets the maximum amount of virtual machine memory

    Virtual machine must be off

  * `virsh setmem --size 2G rhel7`

    It sets the amount of virtual machine memory

    Virtual machine must be running



References:

* [https://www.ibm.com/support/knowledgecenter/en/linuxonibm/com.ibm.linux.z.ldva/ldva_t_modifyingCPUNumber.html](https://www.ibm.com/support/knowledgecenter/en/linuxonibm/com.ibm.linux.z.ldva/ldva_t_modifyingCPUNumber.html)
