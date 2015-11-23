## keepalived

[![Build Status](https://travis-ci.org/Oefenweb/ansible-keepalived.svg?branch=master)](https://travis-ci.org/Oefenweb/ansible-keepalived) [![Ansible Galaxy](http://img.shields.io/badge/ansible--galaxy-keepalived-blue.svg)](https://galaxy.ansible.com/list#/roles/3349)

Set up [Keepalived](http://www.keepalived.org/) in Ubuntu systems.

#### Requirements

None

#### Variables

* `keepalived_ip_nonlocal_bind` [default: `1`]: Allow to bind to IP addresses that are nonlocal, meaning that they're not assigned to a device on the local system
* `keepalived_ip_forward` [default: `1`]: Allow ip forwarding

* `keepalived_global_defs_notification_email` [default: `['root@localhost.localdomain']`]: Email addresses to send alerts to
* `keepalived_global_defs_notification_email_from` [default: `'root@localhost.localdomain'`]: From address that will be in header
* `keepalived_global_defs_smtp_server` [default: `'127.0.0.1'`]: SMTP server IP address
* `keepalived_global_defs_smtp_connect_timeout` [default: `30`]: SMTP server connect timeout in seconds

* `keepalived_vrrp_scripts` [default: `{}`]: VRRP script declarations
* `keepalived_vrrp_scripts.key`: The name of the VRRP script
* `keepalived_vrrp_scripts.key.script`: The script to run periodically
* `keepalived_vrrp_scripts.key.weight`: The check weight to adjust the priority (optional)
* `keepalived_vrrp_scripts.key.interval`: The check interval in seconds (optional)

* `keepalived_vrrp_instances` [default: `{}`]: VRRP instance declarations
* `keepalived_vrrp_instances.key`: The name of the VRRP instance
* `keepalived_vrrp_instances.key.interface`: Interface bound by VRRP
* `keepalived_vrrp_instances.key.state`: Start-up default state (`MASTER|BACKUP`). As soon as the other machine(s) come up, an election will be held and the machine with the highest `priority` will become `MASTER`
* `keepalived_vrrp_instances.key.priority`: For electing `MASTER` highest priority (`0...255`) wins
* `keepalived_vrrp_instances.key.virtual_router_id`: Arbitrary unique number (`0...255`) used to differentiate multiple instances of VRRPD running on the same NIC (and hence same socket)
* `keepalived_vrrp_instances.key.advert_int`: The advert interval in seconds (optional)
* `keepalived_vrrp_instances.key.smtp_alert`: Whether or not to send email notifications during state transitioning (optional)
* `keepalived_vrrp_instances.key.authentication`: Authentication block
* `keepalived_vrrp_instances.key.authentication.auth_type`: Simple password or IPSEC AH (`PASS|AH`)
* `keepalived_vrrp_instances.key.authentication.auth_pass`: Password string (up to 8 characters)
* `keepalived_vrrp_instances.key.virtual_ipaddresses`: VRRP IP address block
* `keepalived_vrrp_instances.key.track_scripts`: Scripts state we monitor
* `keepalived_vrrp_instances.key.notify_backup`: Script to invoke when in backup state
* `keepalived_vrrp_instances.key.notify_master`: Script to invoke when in master state
* `keepalived_vrrp_instances.key.notify_fault`: Script to invoke when in fault state

* `keepalived_virtual_servers`: Dictionary with virtual servers
* `keepalived_virtual_servers.key`: The name of the virtual server, used for docs and grouping
* `keepalived_virtual_servers.key.address`: The virtual IP
* `keepalived_virtual_servers.key.port`: The port to listen on
* `keepalived_virtual_servers.key.delay_loop`: How frequently check the state of the real servers
* `keepalived_virtual_servers.key.lb_algo`: What algorithm to use for load balancing (rr = round robin)
* `keepalived_virtual_servers.key.lb_kind`: Kind of load balancing (DR = direct route, NAT = Network Address Translation)
* `keepalived_virtual_servers.key.protocol`: What protocol to use
* `keepalived_virtual_servers.key.real_servers`: Dictonary with real servers
* `keepalived_virtual_servers.key.real_servers.key`: Name of the real server
* `keepalived_virtual_servers.key.real_servers.key.address`: IP Address of the real server
* `keepalived_virtual_servers.key.real_servers.key.port`: Port Address of the real server
* `keepalived_virtual_servers.key.real_servers.key.persistence_timeout`: How many seconds the make the connection persist to the same real server
* `keepalived_virtual_servers.key.real_servers.key.checks`: Dictionary with checks to perform for the real server
* `keepalived_virtual_servers.key.real_servers.key.checks.key`: Type of check
* `keepalived_virtual_servers.key.real_servers.key.checks.key[0]`: String with check parameters

## Dependencies

None

#### Example

```yaml
---
- hosts: all
  roles:
  - keepalived
```

##### HAProxy

```yaml
keepalived_vrrp_scripts:
  chk_haproxy:
    script: 'killall -0 haproxy'
    weight: 2
    interval: 1
	[rise: 2]
	[fall: 2]

keepalived_vrrp_instances:
  VI_1:
    interface: eth1
    state: MASTER
    priority: 101
    virtual_router_id: 51
	advert_int: 1            # Optional

    authentication:
      auth_type: PASS
      auth_pass: '4Apr3C*d'

    virtual_ipaddresses:
      - '10.0.0.10/24 dev eth1 label eth1:1'

    track_scripts:
      - chk_haproxy
    
	# see http://gcharriere.com/blog/?p=339
    notify_backup: /etc/keepalived/bypass_ipvs.sh add 10.0.0.10    # Optional
    notify_fault: /etc/keepalived/bypass_ipvs.sh add 10.0.0.10     # Optional
    notify_master: /etc/keepalived/bypass_ipvs.sh dell 10.0.0.10   # Optional

keepalived_virtual_servers:
  http:
    address: 10.0.0.10
    port: 80
    delay_loop: 10
    lb_algo: rr
    lb_kind: DR
    protocol: TCP
    persistence_timeout: 5          # Optional
	
    real_servers: 
      lb1:
        address: 10.0.0.20
        port: 80
        checks:
          TCP_CHECK: 
            - 'connect_timeout 3'
      lb2:
        address: 10.0.0.20
        port: 80
        checks:
          TCP_CHECK: 
            - 'connect_timeout 3'
  https:
    address: 10.0.0.10
    port: 443
    delay_loop: 10
    lb_algo: rr
    lb_kind: DR
    protocol: TCP
    persistence_timeout: 5          # Optional
	
    real_servers: 
      lb1:
        address: 10.0.0.20
        port: 443
        checks:
          TCP_CHECK: 
            - 'connect_timeout 3'     # See keepalived docs
      lb2:
        address: 10.0.0.20
        port: 443
        checks:
          TCP_CHECK: 
            - 'connect_timeout 3'
            - 'connect_port 443'
  
```

#### License

MIT

#### Author Information

Mischa ter Smitten

#### Feedback, bug-reports, requests, ...

Are [welcome](https://github.com/Oefenweb/ansible-keepalived/issues)!
