---
title: 【keepalived】Keepalived配置及示例
date: 2019-03-21 09:07:03
tags:
- keepalived
---

### keepalived 简介

Keepalived的作用是检测服务器的状态，如果有一台web服务器宕机，或工作出现故障，Keepalived将检测到，并将有故障的服务器从系统中剔除，同时使用其他服务器代替该服务器的工作，当服务器工作正常后Keepalived自动将服务器加入到服务器群中，这些工作全部自动完成，不需要人工干涉，需要人工做的只是修复故障的服务器。

### keepalived 源码下载地址

[keepalived software](https://www.keepalived.org/software/)

### keepalived 官网

[Keepalived for Linux](https://www.keepalived.org/index.html)

### MySQL、Redis、ActiveMQ 高可用简单配置

```
[root@localhost keepalived]# vi keepalived.conf

! Configuration File for keepalived
global_defs {
   #notification_email {
        #aaa@email.com
   #}
   #notification_email_from keepalived@email.com
   #smtp_server 127.0.0.1
   #smtp_connect_timeout 30
}

vrrp_track_process mysql {
  process "mysqld"
}
vrrp_instance mysql {
    state BACKUP
    nopreempt
    interface ens160
    virtual_router_id 61
    track_process{
		mysql
    }
    virtual_ipaddress {
        192.168.1.2
    }
}

vrrp_track_process redis {
  process "redis-server"
}
vrrp_instance redis {
    state BACKUP
    nopreempt
    interface ens160
    virtual_router_id 62
    track_process{
		redis
    }
    virtual_ipaddress {
        192.168.1.3
    }
    notify_master "/home/admin/redis/bin/redis-cli slaveof no one"
    notify_backup "/home/admin/redis/bin/redis-cli slaveof 192.168.1.191 6379"
}

vrrp_script activemq {
  script "/bin/activemq status"
}
vrrp_instance activemq {
    state BACKUP
    nopreempt
    interface ens160
    virtual_router_id 63
    track_script{
		activemq
    }
    virtual_ipaddress {
        192.168.1.4
    }
}

```

### keepalived 配置手册

```
keepalived.conf(5)     Keepalived Configuration's Manual    keepalived.conf(5)


NAME
       keepalived.conf - configuration file for Keepalived

DESCRIPTION
       keepalived.conf   is   the  configuration  file  which
       describes all the Keepalived keywords. Keywords are placed  in  hierar-
       chies  of  blocks  and subblocks, each layer being delimited by '{' and
       '}' pairs.

       Comments start with '#' or '!' to the end of the  line  and  can  start
       anywhere in a line.

       The  keyword  'include'  allows  inclusion of other configuration files
       from within the main configuration file, or from subsequently  included
       files.

       The format of the include directive is:

       include FILENAME

       FILENAME can be a fully qualified or relative pathname, and can include
       wildcards,   including   csh   style   brace   expressions   such    as
       "{foo/{,cat,dog},bar}" if glob() supports them.

       After  opening  an  included  file, the current directory is set to the
       directory of the file itself, so any relative  paths  included  from  a
       file are relative to the directory of the including file itself.

       Note:  This  documentation  MUST  be considered as THE
       exhaustive source of information in order to configure Keepalived. This
       documenation is supported and maintained by Keepalived Core-Team.

PARAMETER SYNTAX
       <BOOL> is one of on|off|true|false|yes|no
       <TIMER>  is  a  time value in seconds, including
       fractional seconds, e.g. 2.71828 or 3; resolution of  timer  is  micro-
       seconds.

SCRIPTS
       There are three classes of scripts can be configured to be executed.

       (a)  Notify  scripts  that  are  run when a vrrp instance or vrrp group
       changes state, or a virtual server quorum changes between up and down.

       (b) vrrp tracking scripts that will cause vrrp instances to go down  it
       they exit a non-zero exist status, or if a weight is specified will add
       or subtract the weight to/from the priority of that vrrp instance.

       (c) LVS checker misc scripts that will cause a real server to  be  con-
       figured down if they exit with a non-zero status.

       By  default  the  scripts will be executed by user keepalived_script if
       that user exists, or if not by root, but for each script the user/group
       under which it is to be executed can be specified.

       There  are  significant  security  implications if scripts are executed
       with root privileges, especially if the scripts themselves are  modifi-
       able  or  replaceable by a non root user. Consequently, security checks
       are made at startup to ensure that if a script  is  executed  by  root,
       then it cannot be modified or replaced by a non root user.

       All scripts should be written so that they will terminate on receipt of
       a SIGTERM signal. Scripts will be sent SIGTERM if their  parent  termi-
       nates, or it is a script the keepalived is awaiting its exit status and
       it has run for too long.

Quoted strings
       Quoted strings are specified between " characters; more specifically  a
       string  will  only  end  after  a  quoted string if there is whitespace
       afterwards. For example:
              "abcd" efg h jkl "mnop"
       will be the single string "abcd efg h jkl mnop", i.e.  the  embedded  "
       characters are removed.

       Quoted  strings  can  also have escaped characters, like the shell. \a,
       \b, \E, \f, \n, \r, \t, \v, \nnn and \xXX (where nnn is up to  3  octal
       digits,  and  XX is any sequence of hex digits) and \cC (which produces
       the control version of character C) are all supported. \C for any other
       character C is just treated as an escaped version of character C, so \\
       is a \ character and \" will be a " character, but it  won't  start  or
       terminate a quoted string.

       For  specifying  scripts with parameters, unquoted spaces will separate
       the parameters.  If it is required for a parameter to contain a  space,
       it should be enclosed in single quotes (').


CONFIGURATION PARSER
       Traditionally  the  configuration  file  parser has not been one of the
       strengths of keepalived. Lot of efforts have been put to  correct  this
       even if this is not the primal goal of the project.

TOP HIERACHY
       Keepalived configuration file is articulated around a set of configura-
       tion blocks.  Each block is focusing and targetting a  specific  daemon
       family feature. These features are:

       GLOBAL CONFIGURATION

       BFD CONFIGURATION

       VRRPD CONFIGURATION

       LVS CONFIGURATION

GLOBAL CONFIGURATION
       contains  subblocks of Global definitions, Linkbeat interfaces,
       Static track groups,  Static  addresses,  Static  routes,  and
       Static rules

Global definitions
       # Following are global daemon facilities for running
       # keepalived in a separate network namespace:
       # --
       # Set the network namespace to run in.
       # The directory /var/run/keepalived will be created as an
       # unshared mount point, for example for pid files.
       # syslog entries will have _NAME appended to the ident.
       # Note: the namespace cannot be changed on a configuration reload.
       net_namespace NAME

       # ipsets wasn't network namespace aware until Linux 3.13, and so
       # if running with # an earlier version of the kernel, by default
       # use of ipsets is disabled if using a namespace and vrrp_ipsets
       # has not been specified. This options overrides the default and
       # allows ipsets to be used with a namespace on kernels prior to 3.13.
       namespace_with_ipsets

       # If multiple instances of keepalived are run in the same namespace,
       # this will create pid files with NAME as part of the file names,
       # in /var/run/keepalived.
       # Note: the instance name cannot be changed on a configuration reload
       instance NAME

       # Create pid files in /var/run/keepalived
       use_pid_dir

       # Poll to detect media link failure using ETHTOOL, MII or ioctl interface
       # otherwise uses netlink interface.
       linkbeat_use_polling

       # Time for main process to allow for child processes to exit on termination
       # in seconds. This can be needed for very large configurations.
       # (default: 5)
       child_wait_time SECS

       # Global definitions configuration block
       global_defs {
           # Set the process names of the keepalived processes to the default values:
           #   keepalived, keepalived_vrrp, keepalived_ipvs, keepalived_bfd
           process_names

           # Specify the individual process names
	   process_name NAME
	   vrrp_process_name NAME
	   ipvs_process_name NAME
	   bfd_process_name NAME

           # Set of email To: notify
           notification_email {
               admin@example1.com
               ...
           }

           # email from address that will be in the header
           # (default: keepalived@<local host name>)
           notification_email_from admin@example.com

           # Remote SMTP server used to send notification email.
           # IP address or domain name with optional port number.
           # (default port number: 25)
           smtp_server 127.0.0.1 [<PORT>]

           # Name to use in HELO messages.
           # (default: local host name)
           smtp_helo_name <STRING>

           # SMTP server connection timeout in seconds.
           smtp_connect_timeout 30

           # Sets default state for all smtp_alerts
           smtp_alert <BOOL>

           # Sets default state for vrrp smtp_alerts
           smtp_alert_vrrp <BOOL>

           # Sets default state for checker smtp_alerts
           smtp_alert_checker <BOOL>

           # Sets logging all checker failes while checker up
           checker_log_all_failures <BOOL>

	   # If set, keepalived only removes virtual servers at shutdown
	   #  (the kernel will remove the real servers). This is faster
	   #  for large configurations.
	   checker_shutdown_vs_only

           # Don't send smtp alerts for fault conditions
           no_email_faults

           # String identifying the machine (doesn't have to be hostname).
           # (default: local host name)
           router_id <STRING>

           # Multicast Group to use for IPv4 VRRP adverts
           # (default: 224.0.0.18)
           vrrp_mcast_group4 224.0.0.18

           # Multicast Group to use for IPv6 VRRP adverts
           # (default: ff02::12)
           vrrp_mcast_group6 ff02::12

           # sets the default interface for static addresses.
           # (default: eth0)
           default_interface p33p1.3

           # Sync daemon as provided by IPVS kernel code only support
           # a single daemon instance at a time to synchronize connection table.
           # Binding interface, vrrp instance and optional
           #  syncid for lvs syncd
           #  syncid (0 to 255) for lvs syncd
           #  maxlen (1..65507) maximum packet length
           #  port (1..65535) UDP port number to use
           #  ttl (1..255)
           #  group - multicast group address (IPv4 or IPv6)
           # NOTE: maxlen, port, ttl and group are only available on Linux 4.3 or later.
           lvs_sync_daemon <INTERFACE> <VRRP_INSTANCE> [id <SYNC_ID>] [maxlen <LEN>] \
                           [port <PORT>] [ttl <TTL>] [group <IP ADDR>]

           # flush any existing LVS configuration at startup
           lvs_flush

           # flush remaining LVS configuration at shutdown
	   lvs_flush_onstop

           # delay for second set of gratuitous ARPs after transition to MASTER.
           # in seconds, 0 for no second set.
           # (default: 5)
           vrrp_garp_master_delay 10

           # number of gratuitous ARP messages to send at a time after
           # transition to MASTER.
           # (default: 5)
           vrrp_garp_master_repeat 1

           # delay for second set of gratuitous ARPs after lower priority
           # advert received when MASTER.
           vrrp_garp_lower_prio_delay 10

           # number of gratuitous ARP messages to send at a time after
           # lower priority advert received when MASTER.
           vrrp_garp_lower_prio_repeat 1

           # minimum time interval for refreshing gratuitous ARPs while MASTER.
           # in seconds.
           # (default: 0 (no refreshing))
           vrrp_garp_master_refresh 60

           # number of gratuitous ARP messages to send at a time while MASTER
           # (default: 1)
           vrrp_garp_master_refresh_repeat 2

           # Delay in ms between gratuitous ARP messages sent on an interface
           # decimal, seconds (resolution usecs).
           # (default: 0)
           vrrp_garp_interval 0.001

           # Delay in ms between unsolicited NA messages sent on an interface
           # decimal, seconds (resolution usecs).
           # (default: 0)
           vrrp_gna_interval 0.000001

           # By default keepalived sends 5 gratuitions ARP/NA messages at a
           # time, and after transitioning to MASTER sends a second block of
           # 5 messages 5 seconds later.
           # With modern switches this is unnecessary, so setting vrrp_min_garp
           # causes only one ARP/NA message to be sent, with no repeat 5 seconds
           # later.
           vrrp_min_garp [<BOOL>]

           # If a lower priority advert is received, don't send another advert.
           # This causes adherence to the RFCs. Defaults to false, unless
           # strict_mode is set.
           vrrp_lower_prio_no_advert [<BOOL>]

           # If we are master and receive a higher priority advert, send an advert
           # (which will be lower priority than the other master), before we
           # transition to backup. This means that if the other master has
           # garp_lower_priority_repeat set, it will resend garp messages.
           # This is to get around the problem of their having been two simultaneous
           # masters, and the last GARP messages seen were from us.
           vrrp_higher_prio_send_advert [<BOOL>]

           # Set the default VRRP version to use
           # (default: 2 , but IPv6 instances will use version 3)
           vrrp_version <2 or 3>

           # Specify the iptables chain for ensuring a version 3 instance
           # doesn't respond on addresses that it doesn't own.
           # Note: it is necessary for the specified chain to exist in
           # the iptables and/or ip6tables configuration, and for the chain
           # to be called from an appropriate point in the iptables configuration.
           # It will probably be necessary to have this filtering after accepting
           # any ESTABLISHED,RELATED packets, because IPv4 might select the VIP as
           # the source address for outgoing connections.
           # (default: INPUT)
           vrrp_iptables keepalived

           # Use nftables to implement no_accept mode.
           #   TABLENAME must not exist, and must be different for each
           #   instance of keepalived running in the same network namespace.
           #   Default tablename is keepalived, and priority is -1.
           #   keepalived will create base chains in the table.
           #   counters means counters are added to the rules (primarily for
           #   debugging purposes).
           #   ifindex means create IPv6 link local sets using ifindex rather
           #   than ifnames. This is the default unless the vrrp_instance has
           #   set dont_track_primary. The alternative is to use interface names
           #   as part of the set key, but nftables prior to v0.8.3 will then no
           #   longer work.
           nftables [TABLENAME]
           nftables_priority PRIORITY
           nftables_counters
           nftables_ifindex

           # or for outbound filtering as well
           # Note, outbound filtering won't work with IPv4, since the VIP can be
           # selected as the source address for an outgoing connection. With IPv6
           # this is unlikely since the addresses are deprecated.
           vrrp_iptables keepalived_in keepalived_out

           # or to not add any iptables rules:
           vrrp_iptables

           # Keepalived may have the option to use ipsets in conjunction with
           # iptables. If so, then the ipset names can be specified, defaults
           # as below. If no names are specified, ipsets will not be used,
           # otherwise any omitted names will be constructed by adding "_if"
           # and/or "6" to previously specified names.
           vrrp_ipsets [keepalived [keepalived6 [keepalived_if6]]]

           # The following enables checking that when in unicast mode, the
           # source address of a VRRP packet is one of our unicast peers.
           vrrp_check_unicast_src

           # Checking all the addresses in a received VRRP advert can be time
           # consuming. Setting this flag means the check won't be carried out
           # if the advert is from the same master router as the previous advert
           # received.
           # (default: don't skip)
           vrrp_skip_check_adv_addr

           # Enforce strict VRRP protocol compliance. This will prohibit:
           #   0 VIPs
           #   unicast peers
           #   IPv6 addresses in VRRP version 2
           vrrp_strict

	   # Send vrrp instance priority notifications on notify FIFOs.
	   vrrp_notify_priority_changes <BOOL>

           # The following options can be used if vrrp or checker processes
           # are timing out. This can be seen by a backup vrrp instance becoming
           # master even when the master is still running because the master or
           # backup system is too busy to process vrrp packets.
           # --
           # Set the vrrp child process priority (Negative values increase priority)
           vrrp_priority <-20 to 19>

           # Set the checker child process priority
           checker_priority <-20 to 19>

           # Set the BFD child process priority
           bfd_priority <-20 to 19>

           # Set the vrrp child process non swappable
           vrrp_no_swap

           # Set the checker child process non swappable
           checker_no_swap

           # Set the BFD child process non swappable
           bfd_no_swap

           # Set the vrrp child process to use real-time scheduling
           # at the specified priority
           vrrp_rt_priority <1..99>

           # Set the checker child process to use real-time scheduling
           # at the specified priority
           checker_rt_priority <1..99>

           # Set the BFD child process to use real-time scheduling
           # at the specified  priority
           bfd_rt_priority <1..99>

           # Set the limit on CPU time between blocking system calls,
           # in microseconds
           # (default: 1000)
           vrrp_rlimit_rtime >=1
           checker_rlimit_rtime >=1
           bfd_rlimit_rtime >=1

           # If Keepalived has been build with SNMP support, the following
           # keywords are available.
           # Note: Keepalived, checker and RFC support can be individually
           # enabled/disabled
           # --
           # Specify socket to use for connecting to SNMP master agent
           # (see source module keepalived/vrrp/vrrp_snmp.c for more details)
           # (default: unix:/var/agentx/master)
           snmp_socket udp:1.2.3.4:705

           # enable SNMP handling of vrrp element of KEEPALIVED MIB
           enable_snmp_vrrp

           # enable SNMP handling of checker element of KEEPALIVED MIB
           enable_snmp_checker

           # enable SNMP handling of RFC2787 and RFC6527 VRRP MIBs
           enable_snmp_rfc

           # enable SNMP handling of RFC2787 VRRP MIB
           enable_snmp_rfcv2

           # enable SNMP handling of RFC6527 VRRP MIB
           enable_snmp_rfcv3

           # enable SNMP traps
           enable_traps

           # If Keepalived has been build with DBus support, the following
           # keywords are available.
           # --
           # Enable the DBus interface
           enable_dbus

           # Name of DBus service
           # Useful if you want to run multiple keepalived processes with DBus enabled
           # (default: org.keepalived.Vrrp1)
           dbus_service_name SERVICE_NAME

           # Specify the default username/groupname to run scripts under.
           # If this option is not specified, the user defaults to keepalived_script
           # if that user exists, otherwise root.
           # If groupname is not specified, it defaults to the user's group.
           script_user username [groupname]

           # Don't run scripts configured to be run as root if any part of the path
           # is writable by a non-root user.
           enable_script_security

           # Rather than using notify scripts, specifying a fifo allows more
           # efficient processing of notify events, and guarantees that they
           # will be delivered in the correct sequence.
           # NOTE: the FIFO names must all be different
           # --
           # FIFO to write notify events to
           # See vrrp_notify_fifo and lvs_notify_fifo for format of output
	   # For further details, see the description under vrrp_sync_group.
	   # see doc/samples/sample_notify_fifo.sh for sample usage.
           notify_fifo FIFO_NAME [username [groupname]]

           # script to be run by keepalived to process notify events
           # The FIFO name will be passed to the script as the last parameter
           notify_fifo_script STRING|QUOTED_STRING [username [groupname]]

           # FIFO to write vrrp notify events to.
           # The string written will be a line of the form: INSTANCE "VI_1" MASTER 100
           # and will be terminated with a new line character.
           # For further details of the output, see the description under vrrp_sync_group
           # and doc/samples/sample_notify_fifo.sh for sample usage.
           vrrp_notify_fifo FIFO_NAME [username [groupname]]

           # script to be run by keepalived to process vrrp notify events
           # The FIFO name will be passed to the script as the last parameter
           vrrp_notify_fifo_script STRING|QUOTED_STRING [username [groupname]]

           # FIFO to write notify healthchecker events to
           # The string written will be a line of the form:
           # VS [192.168.201.15]:tcp:80 {UP|DOWN}
           # RS [1.2.3.4]:tcp:80 [192.168.201.15]:tcp:80 {UP|DOWN}
           # and will be terminated with a new line character.
           lvs_notify_fifo FIFO_NAME [username [groupname]]

           # script to be run by keepalived to process healthchecher notify events
           # The FIFO name will be passed to the script as the last parameter
           lvs_notify_fifo_script STRING|QUOTED_STRING [username [groupname]]

           # Allow configuration to include interfaces that don't exist at startup.
           # This allows keepalived to work with interfaces that may be deleted and restored
           #   and also allows virtual and static routes and rules on VMAC interfaces.
           #   allow_if_changes allows an interface to be deleted and recreated with a
           #   different type or underlying interface, eg changing from vlan to macvlan
           #   or changing a macvlan from eth1 to eth2. This is predominantly used for
           #   reporting duplicate VRID errors at startup if allow_if_changes is not set.
           dynamic_interfaces [allow_if_changes]

           # The following options are only needed for large configurations, where either
           # keepalived creates a large number of interface, or the system has a large
           # number of interface. These options only need using if
           # "Netlink: Receive buffer overrun" messages are seen in the system logs.
           # If the buffer size needed exceeds the value in /proc/sys/net/core/rmem_max
           #  the corresponding force option will need to be set.
           # --
           # Set netlink receive buffer size. This is useful for
           # very large configurations where a large number of interfaces exist, and
           # the initial read of the interfaces on the system causes a netlink buffer
           # overrun.
           vrrp_netlink_cmd_rcv_bufs BYTES
           vrrp_netlink_cmd_rcv_bufs_force <BOOL>
           vrrp_netlink_monitor_rcv_bufs BYTES
           vrrp_netlink_monitor_rcv_bufs_force <BOOL>

           # The vrrp netlink command and monitor socket the checker command and
           # and monitor socket and process monitor buffer sizes can be independently set.
           # The force flag means to use SO_RCVBUFFORCE, so that the buffer size
           # can exceed /proc/sys/net/core/rmem_max.
           lvs_netlink_cmd_rcv_bufs BYTES
           lvs_netlink_cmd_rcv_bufs_force <BOOL>
           lvs_netlink_monitor_rcv_bufs BYTES
           lvs_netlink_monitor_rcv_bufs_force <BOOL>

           # As a guide for process_monitor_rcv_bufs for 1400 processes terminating
           # simultaneously, 212992 (the default on some systems) is insufficient, whereas
           # 500000 is sufficient.
           process_monitor_rcv_bufs BYTES
           process_monitor_rcv_bufs_force <BOOL>

           # When a socket is opened, the kernel configures the max rx buffer size for
           # the socket to /proc/sys/net/core/rmem_default. On some systems this can be
           # very large, and even generally this can be much larger than necessary.
           # This isn't a problem so long as keepalived is reading all queued data from
           # it's sockets, but if rmem_default was set sufficiently large, and if for
           # some reason keepalived stopped reading, it could consume all system memory.
           # The vrrp_rx_bufs_policy allows configuring of the rx bufs size when the
           # sockets are opened. If the policy is MTU, the rx buf size is configured
           # to the total of interface's MTU * vrrp_rx_bufs_multiplier for each vrrp
           # instance using the socket. Likewise, if the policy is ADVERT, then it is
           # the total of each vrrp instances advert packet size * multiplier.
           # (default: use system default)
           vrrp_rx_bufs_policy [MTU|ADVERT|NUMBER]

           # (default: 3)
           vrrp_rx_bufs_multiplier NUMBER

           # Send notifies at startup for real servers that are starting up
           rs_init_notifies

           # Don't send an email every time a real server checker changes state;
           # only send email when a real server is added or removed
           no_checker_emails

           # The umask to use for creating files. The number can be specified in hex,
           #   octal or decimal. BITS are I{R|W|X}{USR|GRP|OTH}, e.g. IRGRP, separated
           #   by '|'s. The default umask is IWGRP | IWOTH. This option cannot override
           #   the command-line option.
           umask [NUMBER|BITS] 

           # On some systems when bond interfaces are created, they can start
	   # passing traffic and then have a several second gap when they stop
	   # passing traffic inbound. This can mean that if keepalived is started
	   # at boot time, i.e. at the same time as bond interfaces are being
	   # created, keepalived doesn't receive adverts and hence can become master
	   # despite an instance with higher priority sending adverts. This option
	   # specifies a delay in seconds before vrrp instances start up after
           # keepalived starts,
           vrrp_startup_delay 5.5

	   # Specify random seed for ${_RANDOM}, to make configurations repeatable
	   # (default is to use a seed based on the time, so that each time a
	   # different configuration will be generated).
	   random_seed  UNSIGNED_INT
       }

Linkbeat interfaces
       The linkbeat_interfaces block allows specifying which interfaces should
       use polling via MII, Ethtool  or  ioctl  status  rather  than  rely  on
       netlink  status  updates.  This  allows more granular control of global
       definition linkbeat_use_polling.

       This   option   is   preferred   over    the    deprecated    use    of
       linkbeat_use_polling  in  a vrrp_instance block, since
       the  latter  only  allows  using  linkbeat  on  the  interface  of  the
       vrrp_instance  itself,  whereas  track_interface  and
       virtual_ipaddresses and virtual_iproutes may require  monitoring  other
       interfaces, which may need to use linkbeat polling.

       The  default polling type to use is MII, unless that isn't supported in
       which case ETHTOOL is used, and if  that  isn't  supported  then  ioctl
       polling. The preferred type of polling to use can be specified with MII
       or ETHTOOL or IOCTL after the interface name, but if  that  type  isn't
       supported, a supported type will be used.

       The synfax for linkbeat_interfaces is:
           linkbeat_interfaces {
               eth2
               enp2s0 ETHTOOL
           }

Static track groups
       Static  track  groups  are used to allow vrrp instances to track static
       addresses, routes and rules. If a static address/route/rule specifies a
       track  group,  then  if the address/route/rule is deleted and cannot be
       restored, the vrrp instance will transition to fault state.

       The syntax for a track group is:
           track_group GROUP1 {
               group {
                   VI_1
                   VI_2
               }
           }

Static routes/addresses/rules
       Keepalived can configure static addresses,  routes,  and  rules.  These
       addresses  are  NOT  moved  by vrrpd, they stay on the
       machine.  If you already have IPs and routes on your machines and  your
       machines  can ping each other, you don't need this section.  The syntax
       for rules and routes is that same as  for  ip  rule  add/ip  route  add
       (except shorted option names aren't supported due to ambiguities).  The
       track_group specification refers to a named track_group which lists the
       vrrp  instances  which  will  track the address, i.e. if the address is
       deleted the vrrp instances will transition to backup.

       NOTE: since rules without preferences can be added in different  orders
       due  to  vrrp  instances transitioning from master to backup etc, rules
       need to have a preference. If a preference is not specified, keepalived
       will assign one, but it will probably not be what you want.

       The  syntax is the same for virtual addresses and virtual routes. If no
       dev element is specified, it  defaults  to  default_interface  (default
       eth0).   Note:  the broadcast address may be specified as '-' or '+' to
       clear or set the host bits of the address.

       If a route or rule could apply to  either IPv4 or IPv6  it will default
       to IPv4. To force a route/rule to be IPv6, add the keyword "inet6".

           static_ipaddress {
               <IPADDR>[/<MASK>] [brd <IPADDR>] [dev <STRING>] [scope <SCOPE>]
                                 [label <LABEL>] [peer <IPADDR>] [home]
                                 [-nodad] [mngtmpaddr] [noprefixroute]
                                 [autojoin] [track_group GROUP]
               192.168.1.1/24 dev eth0 scope global
               ...
           }

           static_routes {
               192.168.2.0/24 via 192.168.1.100 dev eth0 track_group GROUP1

               192.168.100.0/24 table 6909 nexthop via 192.168.101.1 dev wlan0
                                onlink weight 1 nexthop via 192.168.101.2
                                dev wlan0 onlink weight 2

               192.168.200.0/24 dev p33p1.2 table 6909 tos 0x04 protocol bird
                                scope link priority 12 mtu 1000 hoplimit 100
                                advmss 101 rtt 102 rttvar 103 reordering 104
                                window 105 cwnd 106 ssthresh lock 107 realms
                                PQA/0x14 rto_min 108 initcwnd 109 initrwnd 110
                                features ecn

               2001:470:69e9:1:2::4 dev p33p1.2 table 6909 tos 0x04 protocol
                                    bird scope link priority 12 mtu 1000
                                    hoplimit 100 advmss 101 rtt 102 rttvar 103
                                    reordering 104 window 105 cwnd 106 ssthresh
                                    lock 107 rto_min 108 initcwnd 109
                                    initrwnd 110 features ecn fastopen_no_cookie 1
               ...
           }

           static_rules {
               from 192.168.2.0/24 table 1 track_group GROUP1

               to 192.168.2.0/24 table 1

               from 192.168.28.0/24 to 192.168.29.0/26 table small iif p33p1
                                    oif wlan0 tos 22 fwmark 24/12
                                    preference 39 realms 30/20 goto 40

               to 1:2:3:4:5:6:7:0/112 from 7:6:5:4:3:2::/96 table 6908
                                      uidrange 10000-19999

               to 1:2:3:4:6:6:7:0/112 from 8:6:5:4:3:2::/96 l3mdev protocol 12
                                      ip_proto UDP sport 10-20 dport 20-30
               ...
           }

VRRP track processes
       The configuration block looks like:

           vrrp_track_process <STRING> {
               # process to monitor (with optional parameters)
               process <STRING>|<QUOTED_STRING> [<STRING>|<QUOTED_STRING> ...]

               # If matching parameters, specifies a partial match (i.e. the first
               #   n parameters match exactly, or an initial match, i.e. the last
               #   parameter may be longer that the parameter configured.
               # To specify that a command must have no parameters, don't specify
               #   any parameters, but specify param_match.
               param_match {initial|partial}

               # default weight (default is 1)
               weight <-254..254>

               # minimum number of processes for success
               quorum NUM

               # maximum number of processes for success. For example, setting
               #   this to 1 would cause a failure if two instances of the process
               #   were running (but beware forks - see fork_delay below).
               #   Setting this to 0 would mean failure if the matching process were
               #   running at all.
               quorum_max NUM

	       # time to delay after process quorum gained after fork before
	       #   consider process up (in fractions of second)
	       #   This is to avoid up/down bounce for fork/exec
	       fork_delay
  
               # time to delay after process quorum lost before
	       #   consider process down (in fractions of second)
	       #   This is to avoid down/up bounce after terminate/parent refork.
	       terminate_delay SECS

	       # this sets fork_delay and terminate_delay
               delay SECS

               # Normally process string is matched against the process name,
	       #   as shown on the Name: line in /proc/PID/status, unless
	       #   parameters are specified.
	       #   This option forces matching the full command line
               full_command
           }

       To  avoid  having to frequently run a track_script to monitor the exis-
       tance of processes (often haproxy  or  nginx),  vrrp_track_process  can
       monitor whether other processes are running.

       One difference from pgrep is track_process doesn't do a regular expres-
       sion match of the command string, but does an exact match. 'pgrep  ssh'
       will  match an sshd process, this track_process will not (it is equiva-
       lent to pgrep "^ssh$").

       If full_command is used (equivalent to pgrep -f), /proc/PID/cmdline  is
       used,  but  any  updates  to  cmdline  will  not be detected (a process
       shouldn't normally change it, although it is possible with great  care,
       for example systemd).

       Prior to Linux v3.2 track_process will not support detection of changes
       to a process name, since the kernel did not notify changes  of  process
       name  prior  to  3.2.  Most processes do not change their process name,
       but, for example, firefox forks processes  that  change  their  process
       name  to  "Web  Content". The process name referred to here is the con-
       tents of /proc/PID/comm.

       Quorum  is  the number of matching processes that must be run for an OK
       status.

       Delay might be useful if it anticipated that a process may be  reloaded
       (stopped  and  restarted),  and  it isn't desired to down and up a vrrp
       instance.

       A positive weight means that an OK status will  add  <weight>  to
       the priority of all VRRP instances which monitor it. On the opposite, a
       negative weight will be subtracted from the initial priority in case of
       insufficient processes.

       If  the  vrrp  instance  or sync group is not the address owner and the
       result is between -253 and 253, the result will be added to the initial
       priority  of the VRRP instance (a negative value will reduce the prior-
       ity), although the effective priority will  be  limited  to  the  range
       [1,254].

       If  a  vrrp instance using a track_process is a member of a sync group,
       unless sync_group_tracking_weight is set on the group weight 0 must  be
       set.   Likewise,  if  the  vrrp instance is the address owner, weight 0
       must also be set.

       Rational for not using pgrep/pidof/killall and the likes:

       Every time pgrep or its equivalent  is  run,  it  iterates  though  the
       /proc/[1-9][0-9]*  directories, and opens the status and cmdline pseudo
       files in each directory.  The cmdline pseudo  file  is  mapped  to  the
       process's  address space, and so if that part of the process is swapped
       out, it will have to be fetched from the swap space.   pgrep  etc  also
       include zombie processes whereas keepalived does not, since they aren't
       running.

       This implementation only iterates though /proc/[1-9][0-9]*/ directories
       at  start  up,  and  it  won't  even  read  the cmdline pseudo files if
       'full_command' is not  specified  for  any  of  the  vrrp_track_process
       entries.  After  startup,  it  uses the process_events kernel <->
       userspace connector to receive  notification  of  process  changes.  If
       full_command  is  specified for any track_process instance, the cmdline
       pseudo file will have to be read upon notification of the  creation  of
       the new process, but at that time it is very unlikely that it will have
       already been swapped out.

       On a busy system with a high number of process  creations/terminations,
       using  a  track_script  with pgrep/pidof/killall may be more efficient,
       although those processes are inefficient compared to the  minimum  that
       keepalived needs.

       Using  pgrep  etc  on  a system that is swapping can have a significant
       detrimental impact on the performance of the system, due to  having  to
       fetch  swapped  memory  from the swap space, thereby causing additional
       swapping.

BFD CONFIGURATION
       This  is  an implementation of RFC5880 (Bidirectional forwarding detec-
       tion), and  this  can  be  configured  to  work  between  2  keepalived
       instances, but using unweighted track_bfds between a master/backup pair
       of VRRP instances means that the VRRP instance will  only  be  able  to
       come  up  if both VRRP instance are running, which somewhat defeats the
       purpose of VRRP.

       This  imlpementation  has  been  tested  with  OpenBFDD  (available  at
       https://github.com/dyninc/OpenBFDD).

       The syntax for bfd instance is :

       bfd_instance <STRING> {
           # BFD Neighbor IP (synonym neighbour_ip)
           neighbor_ip <IP ADDRESS>

           # Source IP to use (optional)
           source_ip <IP ADDRESS>

           # Required min RX interval, in ms
           # (default is 10 ms)
           min_rx <INTEGER>

           # Desired min TX interval, in ms
           # (default is 10 ms)
           min_tx <INTEGER>

           # Desired idle TX interval, in ms
           # (default is 1000 ms)
           idle_tx <INTEGER>

           # Number of missed packets after
           # which the session is declared down
           # (default is 5)
           multiplier <INTEGER>

           # Operate in passive mode (default is active)
           passive

           # outgoing IPv4 ttl to use (default 255)
           ttl <INTEGER>

           # outgoing IPv6 hoplimit to use (default 64)
           hoplimit <INTEGER>

           # maximum reduction of ttl/hoplimit
           #  in received packet (default 0)
           #  (255 disables hop count checking)
           max_hops <INTEGER>

           # Default tracking weight
           weight
       }

VRRPD CONFIGURATION
       contains  subblocks  of  VRRP  script(s),  VRRP synchronization
       group(s), VRRP gratuitous ARP and unsolicited  neighbour  advert  delay
       group(s) and VRRP instance(s)

VRRP script(s)
       The  script  will be executed periodically, every <interval> sec-
       onds. Its exit code will be recorded for all VRRP instances which moni-
       tor  it.   Note  that  the script will only be executed if at least one
       VRRP instance monitors it.

       The default weight equals 0, which means that any VRRP  instance  moni-
       toring the script will transition to the fault state after <fall>
       consecutive failures of the script. After that,  <rise>  consecu-
       tive  successes  will  cause  VRRP  instances to leave the fault state,
       unless they are also in the fault state due to other scripts or  inter-
       faces that they are tracking.

       A   positive   weight   means  that  <rise>  successes  will  add
       <weight> to the priority of all VRRP instances which monitor  it.
       On  the opposite, a negative weight will be subtracted from the initial
       priority in case of <fall> failures.

       The syntax for the vrrp script is:

       # Adds a script to be executed periodically. Its exit code will be
       # recorded for all VRRP instances and sync groups which are monitoring it.
       vrrp_script <SCRIPT_NAME> {
           # path of the script to execute
           script <STRING>|<QUOTED-STRING>

           # seconds between script invocations, (default: 1 second)
           interval <INTEGER>

           # seconds after which script is considered to have failed
           timeout <INTEGER>

           # adjust priority by this weight, (default: 0)
           weight <INTEGER:-253..253>

           # required number of successes for OK transition
           rise <INTEGER>

           # required number of successes for KO transition
           fall <INTEGER>

           # user/group names to run script under.
           #  group default to group of user
           user USERNAME [GROUPNAME]

           # assume script initially is in failed state
           init_fail
       }

VRRP track files
       Adds a file to be monitored. The script will be  read  whenever  it  is
       modified. The value in the file will be recorded for all VRRP instances
       and sync groups which monitor it.  Note that the file will only be read
       if at least one VRRP instance or sync group monitors it.

       A  value will be read as a number in text from the file.  If the weight
       configured against the track_file is 0, a non-zero value  in  the  file
       will  be  treated as a failure status, and a zero value will be treaded
       as an OK status, otherwise the value will be  multiplied by the  weight
       configured in the track_file statement. If the result is less than -253
       any VRRP instance or sync group monitoring the script  will  transition
       to the fault state (the weight can be 254 to allow for a negative value
       being read from the file).

       If the vrrp instance or sync group is not the  address  owner  and  the
       result is between -253 and 253, the result will be added to the initial
       priority of the VRRP instance (a negative value will reduce the  prior-
       ity),  although  the  effective  priority  will be limited to the range
       [1,254].

       If a vrrp instance using a track_file is a  member  of  a  sync  group,
       unless  sync_group_tracking_weight is set on the group weight 0 must be
       set.  Likewise, if the vrrp instance is the  address  owner,  weight  0
       must also be set.

       The syntax for vrrp track file is :

       vrrp_track_file <STRING> {    # VRRP track file declaration
           # file to track (weight defaults to 1)
           file <QUOTED_STRING>

           # optional default weight
           weight <-254..254>

           # create the file and/or initialise the value
           # This causes VALUE (default 0) to be written to
           # the specified file at startup if the file doesn't
           # exist, unless overwrite is specified in which case
           # any existing file contents will be overwritten with
           # the specified value.
           init_file [VALUE] [overwrite]
       }

VRRP synchronization group(s)
       VRRP  Sync  Group is an extension to VRRP protocol. The main goal is to
       define a bundle of VRRP instance to get synchronized together  so  that
       transition of one instance will be reflected to others group members.

       In  addition there is an enhanced notify feature for fine state transi-
       tion catching.

       You can also define multiple track policy in order to force state tran-
       sition  according  to  a  third party event such as interface, scripts,
       file, BFD.

       Important: for a SYNC group to  run  reliably,  it  is
       vital  that  all instances in the group are MASTER or that they are all
       either BACKUP or FAULT. A situation with half instances  having  higher
       priority  on  machine  A  half others with higher priority on machine B
       will lead to constant re-elections. For this reason, when instances are
       grouped,   any  track  scripts/files  configured  against  member  VRRP
       instances will have their tracking weights automatically set  to  zero,
       in order to avoid inconsistent priorities across instances.

       The syntax for vrrp_sync_group is :

       vrrp_sync_group <STRING> {
           group {
               # name of the vrrp_instance (see below)
               # Set of VRRP_Instance string
               <STRING>
               <STRING>
               ...
           }

           # Synchronization group tracking interface, script, file & bfd will
           # update the status/priority of all VRRP instances which are members
           # of the sync group.
           track_interface {
               eth0
               eth1
               eth2 weight <-253..253>
               ...
           }

           # add a tracking script to the sync group (<SCRIPT_NAME> is the name
           # of the vrrp_script entry) go to FAULT state if any of these go down
           # if unweighted.
           track_script {
               <SCRIPT_NAME>
               <SCRIPT_NAME> weight <-253..253>
           }

           # Files whose state we monitor, value is added to effective priority.
           # <STRING> is the name of a vrrp_status_file
           # weight defaults to weight configured in vrrp_track_file
           track_file {
               <STRING>
               <STRING> weight <-254..254>
               ...
           }

           # BFD instances we monitor, value is added to effective priority.
           # <STRING> is the name of a BFD instance
           track_bfd {
               <STRING>
               <STRING>
               <STRING> weight <INTEGER: -253..253>
               ...
           }

           # notify scripts and alerts are optional
           #
           # filenames of scripts to run on transitions can be unquoted (if
           # just filename) or quoted (if it has parameters)
           # The username and groupname specify the user and group
           # under which the scripts should be run. If username is
           # specified, the group defaults to the group of the user.
           # If username is not specified, they default to the
           # global script_user and script_group to MASTER transition
           notify_master /path/to_master.sh [username [groupname]]

           # to BACKUP transition
           notify_backup /path/to_backup.sh [username [groupname]]

           # FAULT transition
           notify_fault "/path/fault.sh VG_1" [username [groupname]]

           # executed when stopping vrrp
           notify_stop <STRING>|<QUOTED-STRING> [username [groupname]]

           # for ANY state transition.
           # "notify" script is called AFTER the notify_* script(s) and
           # is executed with 4 additional arguments after the configured
           # arguments provided by Keepalived:
           #   $(n-3) = "GROUP"|"INSTANCE"
           #   $(n-2) = name of the group or instance
           #   $(n-1) = target state of transition (stop only applies to instances)
           #            ("MASTER"|"BACKUP"|"FAULT"|"STOP")
           #   $(n)   = priority value
           #   $(n-3) and $(n-1) are ALWAYS sent in uppercase, and the possible
           #
           # strings sent are the same ones listed above
           #   ("GROUP"/"INSTANCE", "MASTER"/"BACKUP"/"FAULT"/"STOP")
           # (note: STOP is only applicable to instances)
           notify <STRING>|<QUOTED-STRING> [username [groupname]]

           # The notify fifo output is the same as the last 4 parameters for the "notify"
           # script, with the addition of "MASTER_RX_LOWER_PRI" instead of state for an
	   # instance, and also "MASTER_PRIORITY" and "BACKUP_PRIORITY" if the priority
           # changes and notify_priority_changes is configured.
           # MASTER_RX_LOWER_PRI is used if a master needs to set some external state, such
           # as setting a secondary IP address when using Amazon AWS; if another keepalived
           # has transitioned to master due to a communications break, the lower priority
           # instance will have taken over the secondary IP address, and the proper master
           # needs to be able to restore it.

           # Send FIFO notifies for vrrp priority changes
	   notify_priority_changes <BOOL>

           # Send email notification during state transition,
           # using addresses in global_defs above (default no,
           # unless global smtp_alert/smtp_alert_vrrp set)
           smtp_alert <BOOL>

           # DEPRECATED. Use track_interface, track_script and
           # track_file on vrrp_sync_groups instead.
           global_tracking

           # allow sync groups to use differing weights.
           # This probably WON'T WORK, but is a replacement for
           # global_tracking in case different weights were used
           # across different vrrp instances in the same sync group.
           sync_group_tracking_weight
       }

VRRP gratuitous ARP and unsolicited neighbour advert delay group(s)
       specifies  the  setting  of  delays between sending gratuitous ARPs and
       unsolicited neighbour advertisements. This  is  intended  for  when  an
       upstream switch is unable to handle being flooded with ARPs/NAs.

       Use  interface  when the limits apply on the single physical interface.
       Use interfaces when a group of interfaces are linked to the same switch
       and the limits apply to the switch as a whole.

       Note:  Only  one  of interface or interfaces should be
       used per block.

       If the global vrrp_garp_interval and/or vrrp_gna_interval are set,  any
       interfaces  that  aren't  specified  in  a  garp_group will inherit the
       global settings.

       The syntax for garp_group is :

       garp_group {
           # Sets the interval between Gratuitous ARP (in seconds, resolution microseconds)
           garp_interval <DECIMAL>

           # Sets the default interval between unsolicited NA (in seconds, resolution microseconds)
           gna_interval <DECIMAL>

           # The physical interface to which the intervals apply
           interface <STRING>

           # A list of interfaces accross which the delays are aggregated.
           interfaces {
               <STRING>
               <STRING>
               ...
           }
       }

VRRP instance(s)
       A VRRP Instance is the VRRP protocol key feature. It defines  and  con-
       figures  VRRP  behaviour  to  run  on  a  specific interface. Each VRRP
       Instances are related to a uniq interface.

       The syntax for vrrp_instance is :

       vrrp_instance <STRING> {
           # Initial state, MASTER|BACKUP
           # As soon as the other machine(s) come up,
           # an election will be held and the machine
           # with the highest priority will become MASTER.
           # So the entry here doesn't matter a whole lot.
           state MASTER

           # interface for inside_network, bound by vrrp
           interface eth0

           # Use VRRP Virtual MAC.
           # NOTE: If sysctl net.ipv4.conf.all.rp_filter is set,
           # and this vrrp_instance is an IPv4 instance, using
           # this option will cause the individual interfaces to be
           # updated to the greater of their current setting, and
           # all.rp_filter, as will default.rp_filter, and all.rp_filter
           # will be set to 0.
           # The original settings are restored on termination.
           use_vmac [<VMAC_INTERFACE>]

           # Send/Recv VRRP messages from base interface instead of
           # VMAC interface
           vmac_xmit_base

           # force instance to use IPv6 (this option is deprecated since
           # the virtual ip addresses determine whether IPv4 or IPv6 is used).
           native_ipv6

           # Ignore VRRP interface faults (default unset)
           dont_track_primary

           # optional, monitor these as well.
           # go to FAULT state if any of these go down if unweighted.
           # When a weight is specified in track_interface, instead of setting the vrrp
           # instance to the FAULT state in case of failure, its priority will be
           # increased by the weight when the interface is up (for positive weights),
           # or decreased by the weight's absolute value when the interface is down
           # (for negative weights). The weight must be comprised between -254 and +254
           # inclusive. 0 is the default behaviour which means that a failure implies a
           # FAULT state. The common practice is to use positive weights to count a
           # limited number of good services so that the server with the highest count
           # becomes master. Negative weights are better to count unexpected failures
           # among a high number of interfaces, as it will not saturate even with high
           # number of interfaces.
           track_interface {
               eth0
               eth1
               eth2 weight <-253..253>
                ...
           }

           # add a tracking script to the interface
           # (<SCRIPT_NAME> is the name of the vrrp_track_script entry)
           # The same principle as track_interface can be applied to track_script entries,
           # except that an unspecified weight means that the default weight declared in
           # the script will be used (which itself defaults to 0).
           track_script {
               <SCRIPT_NAME>
               <SCRIPT_NAME> weight <-253..253>
           }

           # Files whose state we monitor, value is added to effective priority.
           # <STRING> is the name of a vrrp_track_file
           track_file {
               <STRING>
               <STRING>
               <STRING> weight <-254..254>
               ...
           }

           # BFD instances we monitor, value is added to effective priority.
           # <STRING> is the name of a BFD instance
           track_bfd {
               <STRING>
               <STRING>
               <STRING> weight <INTEGER: -253..253>
               ...
           }

           # default IP for binding vrrpd is the primary IP
           # on interface. If you want to hide the location of vrrpd,
           # use this IP as src_addr for multicast or unicast vrrp
           # packets. (since it's multicast, vrrpd will get the reply
           # packet no matter what src_addr is used).
           # optional
           mcast_src_ip <IPADDR>
           unicast_src_ip <IPADDR>

           # if the configured src_ip doesn't exist or is removed put the
           # instance into fault state
           track_src_ip

           # VRRP version to run on interface
	   # default is global parameter vrrp_version, but IPv6 instances will
	   # always use version 3.
           version <2 or 3>

           # Do not send VRRP adverts over a VRRP multicast group.
           # Instead it sends adverts to the following list of
           # ip addresses using unicast. It can be cool to use
           # the VRRP FSM and features in a networking
           # environment where multicast is not supported!
           # IP addresses specified can be IPv4 as well as IPv6.
           unicast_peer {
               <IPADDR>
               ...
           }

           # The checksum calculation when using VRRPv3 changed after v1.3.6.
           #  Setting this flag forces the old checksum algorithm to be used
           #  to maintain backward compatibility, although keepalived will
           #  attempt to maintain compatibility anyway if it sees an old
           #  version checksum. Specifying never will turn off auto detection
           #  of old checksums. [This option may not be enabled - check output
           #  of `keepalived -v` for OLD_CHKSUM_COMPAT.]
           old_unicast_checksum [never]

           # interface specific settings, same as global parameters.
           # default to global parameters
           garp_master_delay 10
           garp_master_repeat 1
           garp_lower_prio_delay 10
           garp_lower_prio_repeat 1
           garp_master_refresh 60
           garp_master_refresh_repeat 2
           garp_interval 100
           gna_interval 100

           # If a lower priority advert is received, don't send another advert.
           # This causes adherence to the RFCs (defaults to global
           # vrrp_lower_priority_dont_send_advert).
           lower_prio_no_advert [<BOOL>]

           # If we are master and receive a higher priority advert, send an advert
           # (which will be lower priority than the other master), before we transition
           # to backup. This means that if the other master has garp_lower_prio_repeat
           # set, it will resend garp messages. This is to get around the problem of
           # their having been two simultaneous masters, and the last GARP
           # messages seen were from us.
           higher_prio_send_advert [<BOOL>]

           # arbitrary unique number from 0 to 255
           # used to differentiate multiple instances of vrrpd
           # running on the same NIC (and hence same socket).
           virtual_router_id 51

           # for electing MASTER, highest priority wins.
           # to be MASTER, make this 50 more than on other machines.
           priority 100

           # VRRP Advert interval in seconds (e.g. 0.92) (use default)
           advert_int 1

           # Note: authentication was removed from the VRRPv2 specification by
           # RFC3768 in 2004.
           #   Use of this option is non-compliant and can cause problems; avoid
           #   using if possible, except when using unicast, where it can be helpful.
           authentication {
               # PASS|AH
               # PASS - Simple password (suggested)
               # AH - IPSEC (not recommended))
               auth_type PASS

               # Password for accessing vrrpd.
               # should be the same on all machines.
               # Only the first eight (8) characters are used.
               auth_pass 1234
           }

           # addresses add|del on change to MASTER, to BACKUP.
           # With the same entries on other machines,
           # the opposite transition will be occurring.
           # For virutal_ipaddress, virtual_ipaddress_excluded,
           #   virtual_routes and virtual_rules most of the options
           #   match the options of the command ip address/route/rule add.
           #   The track_group option only applies to static addresses/routes/rules.
           #   no_track is specific to keepalived and means that the
           #   vrrp_instance will not transition out of master state
           #   if the address/route/rule is deleted and the address/route/rule
           #   will not be reinstated until the vrrp instance next transitions
           #   to master.
           # <LABEL>: is optional and creates a name for the alias.
                      For compatibility with "ifconfig", it should
                      be of the form <realdev>:<anytext>, for example
                      eth0:1 for an alias on eth0.
           # <SCOPE>: ("site"|"link"|"host"|"nowhere"|"global")
           virtual_ipaddress {
               <IPADDR>[/<MASK>] [brd <IPADDR>] [dev <STRING>] [scope <SCOPE>]
                                 [label <LABEL>] [peer <IPADDR>] [home]
                                 [-nodad] [mngtmpaddr] [noprefixroute]
                                 [autojoin] [no_track]
               192.168.200.17/24 dev eth1
               192.168.200.18/24 dev eth2 label eth2:1
           }

           # VRRP IP excluded from VRRP optional.
           # For cases with large numbers (eg 200) of IPs
           # on the same interface. To decrease the number
           # of addresses sent in adverts, you can exclude
           # most IPs from adverts.
           # The IPs are add|del as for virtual_ipaddress.
           # Can also be used if you want to be able to add
           # a mixture of IPv4 and IPv6 addresses, since all
           # addresses in virtual_ipaddress must be of the
           # same family.
           virtual_ipaddress_excluded {
               <IPADDR>[/<MASK>] [brd <IPADDR>] [dev <STRING>] [scope <SCOPE>]
                                 [label <LABEL>] [peer <IPADDR>] [home]
                                 [-nodad] [mngtmpaddr] [noprefixroute]
                                 [autojoin] [no_track]
               <IPADDR>[/<MASK>] ...
               ...
           }

           # Set the promote_secondaries flag on the interface to stop other
           # addresses in the same CIDR being removed when 1 of them is removed
           # For example if 10.1.1.2/24 and 10.1.1.3/24 are both configured on an
           # interface, and one is removed, unless promote_secondaries is set on
           # the interface the other address will also be removed.
	   prompte_secondaries

           # routes add|del when changing to MASTER, to BACKUP.
           # See static_routes for more details
           virtual_routes {
               # src <IPADDR> [to] <IPADDR>/<MASK> via|gw <IPADDR>
               #   [or <IPADDR>] dev <STRING> scope <SCOPE> table <TABLE>
               src 192.168.100.1 to 192.168.109.0/24 via 192.168.200.254 dev eth1
               192.168.110.0/24 via 192.168.200.254 dev eth1
               192.168.111.0/24 dev eth2 no_track
               192.168.112.0/24 via 192.168.100.254
               192.168.113.0/24 via 192.168.200.254 or 192.168.100.254 dev eth1
               blackhole 192.168.114.0/24
               0.0.0.0/0 gw 192.168.0.1 table 100  # To set a default gateway into table 100.
           }

           # rules add|del when changing to MASTER, to BACKUP
           # See static_rules for more details
           virtual_rules {
               from 192.168.2.0/24 table 1
               to 192.168.2.0/24 table 1 no_track
           }

           # VRRPv3 has an Accept Mode to allow the virtual router when not the
           # address owner to receive packets addressed to a VIP. This is the default
           # setting unless strict mode is set. As an extension, this also works for
           # VRRPv2 (RFC 3768 doesn't define an accept mode).
           # --
           # Accept packets to non address-owner
           accept

           # Drop packets to non address-owner.
           no_accept

           # VRRP will normally preempt a lower priority machine when a higher priority
           # machine comes online.  "nopreempt" allows the lower priority machine to
           # maintain the master role, even when a higher priority machine comes back
           # online.
           # NOTE: For this to work, the initial state of this
           # entry must be BACKUP.
           # --
           nopreempt

           # for backwards compatibility
           preempt

           # See description of global vrrp_skip_check_adv_addr, which
           # sets the default value. Defaults to vrrp_skip_check_adv_addr
           skip_check_adv_addr [on|off|true|false|yes|no]

           # See description of global vrrp_strict
           # If vrrp_strict is not specified, it takes the value of vrrp_strict
           # If strict_mode without a parameter is specified, it defaults to on
           strict_mode [on|off|true|false|yes|no]

           # Seconds after startup or seeing a lower priority master until preemption
           # (if not disabled by "nopreempt").
           # Range: 0 (default) to 1000 (e.g. 4.12)
           # NOTE: For this to work, the initial state of this
           # entry must be BACKUP.
           preempt_delay 300    # waits 5 minutes

           # Debug level, not implemented yet.
           # LEVEL is a number in the range 0 to 4
           debug <LEVEL>

           # notify scripts, alert as above
           notify_master <STRING>|<QUOTED-STRING> [username [groupname]]
           notify_backup <STRING>|<QUOTED-STRING> [username [groupname]]
           notify_fault <STRING>|<QUOTED-STRING> [username [groupname]]
           # executed when stopping vrrp
           notify_stop <STRING>|<QUOTED-STRING> [username [groupname]]
           notify <STRING>|<QUOTED-STRING> [username [groupname]]

           # The notify_master_rx_lower_pri script is executed if a master
           #  receives an advert with priority lower than the master's advert.
           notify_master_rx_lower_pri <STRING>|<QUOTED-STRING> [username [groupname]]

           # Send vrrp instance priority notifications on notify FIFOs.
	   notify_priority_changes <BOOL>

           # Send SMTP alerts
           smtp_alert <BOOL>

           # Set socket receive buffer size (see global_defs
           # vrrp_rx_bufs_policy for explanation)
           kernel_rx_buf_size

           # Set use of linkbeat for the interface of this VRRP instance. This option is
           # deprecated - use linkbeat_interfaces block instead.
	   linkbeat_use_polling
       }

LVS CONFIGURATION
       contains subblocks  of  Virtual  server  group(s)  and
       Virtual server(s)

       The  subblocks  contain arguments for configuring Linux IPVS (LVS) fea-
       ture.  Knowledge of ipvsadm(8) will be helpful here. Configuring LVS is
       achieved by defining virtual server groups, virtual servers and option-
       ally SSL configuration. Every virtual server  defines  a  set  of  real
       servers,  you can attach healthcheckers to each real server. Keepalived
       will then lead LVS operation by dynamically maintaining topology.

       For details of what  configuration  combinations  are  valid,  see  the
       ipvsadm(8) man page.

       Note: Where an option can be configured for a  virtual
       server,  real  server, and possibly checker, the virtual server setting
       is the default for real servers, and the real  server  setting  is  the
       default for checkers.

       Note: Tunnelled real/sorry servers can differ from the
       address family of the  virtual  server  and  non  tunnelled  real/sorry
       servers,  which  all  have  to  be the same. If a virtual server uses a
       fwmark, and all the real/sorry servers are tunnelled, the address  fam-
       ily of the virtual server will be the same as the address family of the
       real/sorry servers if they are all the same, otherwise it will  default
       to IPv4 (use ip_family inet6 to override this).

       Note: The port for the  virtual  server  can  only  be
       omitted if the virtual service is persistent.

Virtual server group(s)
       This feature offers a way to simplify your configuration by factorizing
       virtual server definitions. If you need to define a  bunch  of  virtual
       servers  with  exactly  the same real server topology then this feature
       will make your configuration  much  more  readable  and  will  optimize
       healthchecking  task by only spawning one healthchecking where multiple
       virtual server declaration will spawn  a  dedicated  healthchecker  for
       every real server which will waste system resources.

       Any  combination  of IP addresses, IP address ranges and firewall marks
       can be used. Use of this option is intended for very large LVSs.

       The syntax for virtual_server_group is :

       virtual_server_group <STRING> {
           # Virtual IP Address and Port
           <IPADDR> [<PORT>]
           <IPADDR> [<PORT>]
           ...
           # <IPADDR RANGE> has the form
           # XXX.YYY.ZZZ.WWW-VVV eg 192.168.200.1-10
           # range includes both .1 and .10 address
           <IPADDR RANGE> [<PORT>] # VIP range [VPORT]
           <IPADDR RANGE> [<PORT>]
           ...
           # Firewall Mark (fwmark)
           fwmark <INTEGER>
           fwmark <INTEGER>
           ...
       }

Virtual server(s)
       A virtual_server can be a declaration of one of  <IPADDR>
       [<PORT>]  ,  fwmark  <INTEGER> or
       group <STRING>

       The syntax for virtual_server is :

       virtual_server <IPADDR> [<PORT>]  |
       virtual_server fwmark <INTEGER> |
       virtual_server group <STRING> {
           # LVS scheduler
           lvs_sched rr|wrr|lc|wlc|lblc|sh|mh|dh|fo|ovf|lblcr|sed|nq

           # Enable hashed entry
           hashed
           # Enable flag-1 for scheduler (-b flag-1 in ipvsadm)
           flag-1
           # Enable flag-2 for scheduler (-b flag-2 in ipvsadm)
           flag-2
           # Enable flag-3 for scheduler (-b flag-3 in ipvsadm)
           flag-3
           # Enable sh-port for sh scheduler (-b sh-port in ipvsadm)
           sh-port
           # Enable sh-fallback for sh scheduler  (-b sh-fallback in ipvsadm)
           sh-fallback
           # Enable mh-port for mh scheduler (-b mh-port in ipvsadm)
           mh-port
           # Enable mh-fallback for mh scheduler  (-b mh-fallback in ipvsadm)
           mh-fallback
           # Enable One-Packet-Scheduling for UDP (-O in ipvsadm)
           ops

           # Default LVS forwarding method
           lvs_method NAT|DR|TUN
           # LVS persistence engine name
           persistence_engine <STRING>
           # LVS persistence timeout in seconds, default 6 minutes
           persistence_timeout [<INTEGER>]
           # LVS granularity mask (-M in ipvsadm)
           persistence_granularity <NETMASK>
           # L4 protocol
           protocol TCP|UDP|SCTP
           # If VS IP address is not set,
           # suspend healthchecker's activity
           ha_suspend

           # Send email notification during quorum up/down transition,
           # using addresses in global_defs above (default no,
           # unless global smtp_alert/smtp_alert_checker set)
           smtp_alert <BOOL>

           # Default VirtualHost string for HTTP_GET or SSL_GET
           # eg virtualhost www.firewall.loc
           # Overridden by virtualhost config of real server or checker
           virtualhost <STRING>

           # On daemon startup assume that all RSs are down
           # and healthchecks failed. This helps to prevent
           # false positives on startup. Alpha mode is
           # disabled by default.
           alpha

           # On daemon shutdown consider quorum and RS
           # down notifiers for execution, where appropriate.
           # Omega mode is disabled by default.
           omega

           # Minimum total weight of all live servers in
           # the pool necessary to operate VS with no
           # quality regression. Defaults to 1.
           quorum <INTEGER>

           # Tolerate this much weight units compared to the
           # nominal quorum, when considering quorum gain
           # or loss. A flap dampener. Defaults to 0.
           hysteresis <INTEGER>

           # Script to execute when quorum is gained.
           quorum_up <STRING>|<QUOTED-STRING> [username [groupname]]

           # Script to execute when quorum is lost.
           quorum_down <STRING>|<QUOTED-STRING> [username [groupname]]

           # IP family for a fwmark service (optional)
           ip_family inet|inet6

           # setup realserver(s)

           # RS to add to LVS topology when the quorum isn't achieved.
           #  If a sorry server is configured, all real servers will
           #  be brought down when the quorum is not achieved.
           sorry_server <IPADDR> [<PORT>]
           # applies inhibit_on_failure behaviour to the sorry_server
           sorry_server_inhibit
           # Sorry server LVS forwarding method
           sorry_server_lvs_method NAT|DR|TUN

           # Optional connection timeout in seconds.
           # The default is 5 seconds
           connect_timeout <TIMER>

           # Retry count to make additional checks if check
           # of an alive server fails. Default: 1 unless specified below
           retry <INTEGER>

           # delay before retry after failure
           delay_before_retry <TIMER>

           # Optional random delay to start the initial check
           # for maximum N seconds.
           # Useful to scatter multiple simultaneous
           # checks to the same RS. Enabled by default, with
           # the maximum at delay_loop. Specify 0 to disable
           warmup <TIMER>

           # delay timer for checker polling
           delay_loop <TIMER>

           # Set weight to 0 when healthchecker detects failure
           inhibit_on_failure

           # one entry for each realserver
           real_server <IPADDR> [<PORT>] {
               # relative weight to use, default: 1
               weight <INTEGER>
               # LVS forwarding method
               lvs_method NAT|DR|TUN

               # Script to execute when healthchecker
               # considers service as up.
               notify_up <STRING>|<QUOTED-STRING> [username [groupname]]
               # Script to execute when healthchecker
               # considers service as down.
               notify_down <STRING>|<QUOTED-STRING> [username [groupname]]

               # maximum number of connections to server
               uthreshold <INTEGER>
               # minimum number of connections to server
               lthreshold <INTEGER>

               # Send email notification during state transition,
               # using addresses in global_defs above (default yes,
               # unless global smtp_alert/smtp_alert_checker set)
               smtp_alert <BOOL>

               # Default VirtualHost string for HTTP_GET or SSL_GET
               # eg virtualhost www.firewall.loc
               # Overridden by virtualhost config of a checker
               virtualhost <STRING>

               alpha <BOOL>                    # see above
	       connect_timeout <TIMER>         # see above
               retry <INTEGER>                 # see above
               delay_before_retry <TIMER>      # see above
               warmup <TIMER>                  # see above
               delay_loop <TIMER>              # see above
               inhibit_on_failure <BOOL>       # see above
               log_all_failures <BOOL>         # log all failures when checker up

               # healthcheckers. Can be multiple of each type
               # HTTP_GET|SSL_GET|TCP_CHECK|SMTP_CHECK|DNS_CHECK|MISC_CHECK|BFD_CHECK

               # All checkers have the following options, except MISC_CHECK
               # which only has options alpha onwards, and BFD_CHECK which has none
               # of the standard options:
               CHECKER_TYPE {
                   # ======== generic connection options
                   # Optional IP address to connect to.
                   # The default is the realserver IP
                   connect_ip <IPADDR>

                   # Optional port to connect to
                   # The default is the realserver port
                   connect_port <PORT>

                   # Optional address to use to
                   # originate the connection
                   bindto <IPADDR>

                   # Optional interface to use; needed if
                   # the bindto address is IPv6 link local
                   bind_if <IFNAME>

                   # Optional source port to
                   # originate the connection from
                   bind_port <PORT>

                   # Optional fwmark to mark all outgoing
                   # checker packets with
                   fwmark <INTEGER>

                   alpha <BOOL>                    # see above
		   connect_timeout <TIMER>         # see above
                   retry <INTEGER>                 # see above
                   delay_before_retry <TIMER>      # see above
                   warmup <TIMER>                  # see above
                   delay_loop <TIMER>              # see above
                   inhibit_on_failure <BOOL>       # see above
               }

               # The following options are additional checker specific

               # HTTP and SSL healthcheckers
               HTTP_GET|SSL_GET {
                   # An url to test
                   # can have multiple entries here
                   url {
                     #eg path / , or path /mrtg2/
                     path <STRING>
                     # healthcheck needs status_code
                     # or status_code and digest
                     # Digest computed with genhash
                     # eg digest 9b3a0c85a887a256d6939da88aabd8cd
                     digest <STRING>
                     # status code returned in the HTTP header
                     # eg status_code 200. Default is any 2xx value
                     status_code <INTEGER>
                     # VirtualHost string. eg virtualhost www.firewall.loc
                     # If not set, uses virtualhost from real or virtual server
                     virtualhost <STRING>
                     # Regular expression to search returned data against.
                     # A failure to match causes the check to fail.
                     regex <STRING>
                     # Reverse the sense of the match, so a match of the
                     # returned text causes the check to fail.
                     regex_no_match
                     # Space separated list of options for regex.
                     #  See man pcre2api for a description of the options.
                     #  The following option are supported:
                     #   allow_empty_class alt_bsux auto_callout caseless
                     #   dollar_endonly dotall dupnames extended firstline
                     #   match_unset_backref multiline never_ucp never_utf
                     #   no_auto_capture no_auto_possess no_dotstar_anchor
                     #   no_start_optimize ucp ungreedy utf never_backslash_c
                     #   alt_circumflex alt_verbnames use_offset_limit
                     regex_options OPTIONS 
                     # For complicated regular expressions a larger stack
                     #   may be needed, and this allows the start and maximum
                     #   sizes in bytes to be specified. For more details see
                     #   the documentation for pcre2_jit_stack_create()
                     regex_stack <START> <MAX>
                     # The minimum offset into the returned data to start
                     #   checking for the regex pattern match. This can save
                     #   processing time if the returned data is large.
                     regex_min_offset <OFFSET>
                     # The maximum offset into the returned data for the
                     #   start of the subject match.
                     regex_max_offset <OFFSET>
                   }
               }

               SSL_GET {
                   # when provided, send Server Name Indicator during SSL handshake
                   enable_sni
               }

               # TCP healthchecker
               TCP_CHECK {
                   # No additional options
               }

               # SMTP healthchecker
               SMTP_CHECK {
                   # Optional string to use for the SMTP HELO request
                   helo_name <STRING>|<QUOTED-STRING>
               }

               # DNS healthchecker
               DNS_CHECK {
                   # The retry default is 3.

                   # DNS query type
                   #   A|NS|CNAME|SOA|MX|TXT|AAAA
                   # The default is SOA
                   type <STRING>

                   # Domain name to use for the DNS query
                   # The default is . (dot)
                   name <STRING>
               }

               # MISC healthchecker, run a program
               MISC_CHECK {
                   # The retry default is 0.

                   # External script or program
                   misc_path <STRING>|<QUOTED-STRING>
                   # Script execution timeout
                   misc_timeout <INTEGER>

                   # If set, the exit code from healthchecker is used
                   # to dynamically adjust the weight as follows:
                   #   exit status 0: svc check success, weight
                   #     unchanged.
                   #   exit status 1: svc check failed.
                   #   exit status 2-255: svc check success, weight
                   #     changed to 2 less than exit status.
                   #   (for example: exit status of 255 would set
                   #     weight to 253)
                   # NOTE: do not have more than one dynamic MISC_CHECK per real_server.
                   misc_dynamic

                   # Specify the username/groupname that the script should
                   #   be run under.
                   # If GROUPNAME is not specified, the group of the user
                   #   is used
                   user USERNAME [GROUPNAME]
               }

               # BFD instance name to check
               BFD_CHECK {
                   name <STRING>
               }
           }
       }

       # Parameters used for SSL_GET check.
       # If none of the parameters are specified, the SSL context
       # will be auto generated.
       SSL {
           # Password
           password <STRING>
           # CA file
           ca <STRING>
           # Certificate file
           certificate <STRING>
           # Key file
           key <STRING>
       }

ADVANCED CONFIGURATION
       Configuration  parser  has  been  extended to support advanced features
       such as conditional configuration  and  parameter  substitution.  These
       features are very usefull for any scripted env where configuration tem-
       plate are generated (datacenters).

Conditional configuration and configuration id
       The config-id defaults to the first part of the node name  as  returned
       by uname, and can be overridden with the -i or --config-id command line
       option.

       Any configuration line starting with '@' is a conditional configuration
       line.   The word immediately following (i.e. without any space) the '@'
       character is compared against the config-id, and if they  don't  match,
       the configuration line is ignored.

       Alternatively,  '@^'  is  a negative comparison, so if the word immedi-
       ately following does NOT match the config-id, the configuration line IS
       included.

       The  purpose of this is to allow a single configuration file to be used
       for multiple systems, where the only differences are likely to  be  the
       router_id,  vrrp  instance priorities, and possibly interface names and
       unicast addresses.

       For example:

           global_defs {
               @main   router_id main_router
               @backup router_id backup_router
           }
           ...
           vrrp_instance VRRP {
               ...
               @main    unicast_src_ip 1.2.3.4
               @backup  unicast_src_ip 1.2.3.5
               @backup2 unicast_src_ip 1.2.3.6
               unicast_peer {
                   @^main    1.2.3.4
                   @^backup  1.2.3.5
                   @^backup2 1.2.3.6
               }
               ...
           }

       If keepalived is invoked with -i main, then the router_id will  be  set
       to  main_router,  if invoked with -i backup, then backup_router, if not
       invoked with -i, or with -i anything else, then the router_id will  not
       be set. The unicast peers for main will be 1.2.3.5 and 1.2.3.6.

Parameter substitution
       Substitutable  parameters  can  be specified. The format for defining a
       parameter is:

       $PARAMETER=VALUE

       where there must be no space before the '='  and  only  whitespace  may
       preceed to '$'.  Empty values are allowed.

       Parameter  names  can be made up of any combination of A-Za-z0-9 and _,
       but cannot start with a digit. Parameter names starting with an  under-
       score  should  be considered reserved names that keepalived will define
       for various pre-defined options.

       After a parameter is defined, any occurrence of $PARAMETER followed  by
       whitespace,  or  any occurrence of ${PARAMETER} (which need not be fol-
       lowed by whitespace) will be replaced by VALUE.

       Replacement is recursive, so that if a parameter value itself  includes
       a replaceable parameter, then after the first substitution, the parame-
       ter in the value will then be replaced; the  substitution  is  done  at
       replacement time and not at definition time, so for example:

           $ADDRESS_BASE=10.2.${ADDRESS_BASE_SUB}
           $ADDRESS_BASE_SUB=0
           ${ADDRESS_BASE}.100/32
           $ADDRESS_BASE_SUB=10
           ${ADDRESS_BASE}.100/32

           will produce:
               10.2.0.100/32
               10.2.10.100/32

       Note   in   the  above  examples  the  use  of  both  ADDRESS_BASE  and
       ADDRESS_BASE_SUB required braces ({}) since  the  parameters  were  not
       followed  by  whitespace  (after  the first substitution which produced
       10.2.${ADDRESS_BASE_SUB}.100/32 the parameter is still not followed  by
       whitespace).

       If  a  parameter is not defined, it will not be replaced at all, so for
       example ${UNDEF_PARAMETER} will remain in the configuration  if  it  is
       undefined;  this  means that existing configuration that contains a '$'
       character (for example in a script definition) will not be  changed  so
       long as no new parameter definitions are added to the configuration.

       Parameter substitution works in conjunction with conditional configura-
       tion.  For example:

           @main $PRIORITY=240
           @backup $PRIORITY=200
           ...
           vrrp_instance VI_0 {
               priority $PRIORITY
           }

           will produce:
               ...
               vrrp_instance VI_0 {
                   priority 240
               }
               if the config_id is main.

           $IF_MAIN=@main
           $IF_MAIN priority 240

           will produce:
               priority 240
               if the config_id is main and nothing if the config_id is not main,
               although why anyone would want to use this rather than simply the
               following is not known (but still possible):
                   @main priority 240

       Multiline definitions are also supported, but when used there  must  be
       nothing on the line after the parameter name. A multiline definition is
       specified by ending each line except the last with a '\' character.

       Example:
           $INSTANCE= \
           vrrp_instance VI_${NUM} { \
               interface eth0.${NUM} \
               use_vmac vrrp${NUM}.1 \
               virtual_router_id 1 \
               @high priority 130 \
               @low priority 120 \
               advert_int 1 \
               virtual_ipaddress { \
                   10.0.${NUM}.254/24 \
               } \
               track_script { \
                   offset_instance_${NUM} \
               } \
           }

           $NUM=0
           $INSTANCE

           $NUM=1
           $INSTANCE

       The use of multiline definitions can be nested.

       Example:
           $RS= \
           real_server 192.168.${VS_NUM}.${RS_NUM} 80 { \
               weight 1 \
               inhibit_on_failure \
               smtp_alert \
               MISC_CHECK { \
                   misc_path "${_PWD}/scripts/vs.sh RS_misc.${INST}.${VS_NUM}.${RS_NUM}.0 10.0.${VS_NUM}.4:80->192.168.${VS_NUM}.${RS_NUM}:80" \
               } \

               MISC_CHECK { \
                   misc_path "${_PWD}/scripts/vs.sh RS_misc.${INST}.${VS_NUM}.${RS_NUM}.1 10.0.${VS_NUM}.4:80->192.168.${VS_NUM}.${RS_NUM}:80" \
               } \

               notify_up "${_PWD}/scripts/notify.sh RS_notify.${INST}.${VS_NUM}.${RS_NUM} UP 10.0.${VS_NUM}.4:80->192.168.${VS_NUM}.${RS_NUM}:80" \

               notify_down "${_PWD}/scripts/notify.sh RS_notify.${INST}.${VS_NUM}.${RS_NUM} DOWN 10.0.${VS_NUM}.4:80->192.168.${VS_NUM}.${RS_NUM}:80" \

           }

           $VS= \
           virtual_server 10.0.${VS_NUM}.4 80 { \
               quorum 2 \
               quorum_up "${_PWD}/scripts/notify.sh VS_notify.${INST} UP 10.0.${VS_NUM}.4:80" \
               quorum_down "${_PWD}/scripts/notify.sh VS_notify.${INST} DOWN 10.0.${VS_NUM}.4:80" \
               $RS_NUM=1 \
               $RS \
               $RS_NUM=2 \
               $RS \
               $RS_NUM=3 \
               $RS \
           }

           $VS_NUM=0
           $ALPHA=alpha
           $VS

           $VS_NUM=1
           $ALPHA=
           $VS

       The above will create 2 virtual servers, each with 3 real servers

Pre-defined definitions
       The following pre-defined definitions are defined:

       ${_PWD} : The directory of the  current  configuration
       file (this can be changed if using the include directive).
       ${_INSTANCE} : The instance name (as defined by the -i
       option, defaults to hostname).
       ${_RANDOM  [MIN [MAX]]} : This is replaced by a random
       integer in the range [MIN, MAX], where MIN and MAX  are  optional  non-
       negative integers. Defaults are MIN=0 and MAX=32767.

       Additional pre-defined definitions will be added as their need is iden-
       tified.   It  will  normally be quite straightforward to add additional
       pre-defined definitions, so if you need one, or have a  good  idea  for
       one,          then          raise          an          issue         at
       https://github.com/acassen/keepalived/issues requesting it.

Sequence blocks
       A line starting ~SEQ(var, start, step, end) will cause the remainder of
       the line to be processed multiple times, with  the  variable  $var  set
       initially  to  start, and then $var will be incremented by step repeat-
       edly, terminating when it is greater than end. step may be omitted,  in
       which  case it defaults to 1 or -1, depending on whether end is greater
       or less than start. start  may  also  be  omitted,  in  which  case  it
       defaults to 1 if end > 0 or -1 if end < 0. so, for example:

           ~SEQ(SUBNET, 0, 3) ip_address 10.0.$SUBNET.1

           would produce:
               ip_address 10.0.0.1
               ip_address 10.0.1.1
               ip_address 10.0.2.1
               ip_address 10.0.3.1

       There can be multiple ~SEQ elements on a line, so for example:

           $VI4= \
           vrrp_track_file offset_instance_4.${IF}.${NUM}.${ID} { \
               file "${_PWD}/679/track_files/4.${IF}.${NUM}.${ID}" \
               weight -100 \
           } \
           vrrp_instance vrrp4.${IF}.${NUM}.${ID} { \
               interface bond${IF}.${NUM} \
               use_vmac vrrp4.${IF}.${NUM}.${ID} \
               virtual_router_id ${ID} \
               priority 130 \
               virtual_ipaddress { \
                   10.${IF}.${NUM}.${ID}/24 \
               } \
               track_file { \
                   offset_instance_4.${IF}.${NUM}.${ID} \
               } \
           }

           ~SEQ(IF,0,7) ~SEQ(NUM,0,31) ~SEQ(ID,1,254) $VI4

           will produce 65024 vrrp instances with names from vrrp4.0.0.1 through to
           vrrp4.7.31.254.


AUTHORS
       Initial by Joseph Mack. Extensive updates by Alexandre Cassen & Quentin
       Armitage.

SEE ALSO
       ipvsadm(8), ip --help.



Keepalived                        2018-11-08                keepalived.conf(5)
```

