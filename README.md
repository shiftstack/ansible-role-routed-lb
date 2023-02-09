ansible-role-routed-lb
======================

[![ansible-lint](https://github.com/shiftstack/ansible-role-routed-lb/actions/workflows/lint.yml/badge.svg)](https://github.com/shiftstack/ansible-role-routed-lb/actions/workflows/lint.yml)

This will deploy an advanced Load-Balancer capable of managing routed VIPs with FRR (using BGP) and load-balance traffic with HAproxy.

So let's say, you're hosting a web service that is exposed by one or multiple virtual IP(s) called VIPs (note that it's not required).
You want this VIP to be routed in your infrastructure thanks to BGP protocol.

This role will do the following:
* If BGP neighbors are provided in the config, it'll deploy FRR and peer with your BGP neighbor(s). If the VIPs are created on the node, they'll be routed in your infrastructure.
* Deploy HAproxy to load-balance and monitor your service.
* If the VIPs are provided in the config, they will be created if a minimum number of backend(s) are found healthy for a given service, and therefore routed in BGP if FRR is deployed.
* They will be removed if no backend was found healthy for a given service, therefore not routed in BGP if FRR is deployed

So if you're hosting multiple Load-Balancers, your web traffic will be:
* routed thanks to BGP if FRR is deployed
* load-balanced and high-availability at the VIP level thanks to BGP if FRR is deployed
* load-balanced between healthy backends thanks to HAproxy


Requirements
------------

For now, we test this module on CentOS 9 Stream.

Installation
------------

```
ansible-galaxy install emilienm.routed_lb
ansible-galaxy collection install ansible.posix
```


Role Variables
--------------

Only `configs` needs to be set, as a dictionary:

```
configs:
  lb1:
    bgp_asn: <BGP ASN>
    bgp_neighbors: # optional, when not set FRR won't be deployed
      - ip: <IP of a BGP router>
        password: <BGP password>
    services:
      - name: <Service name (e.g. api)>
        vips:
          - <VIPs that are used to reach that service in frontend>
        min_backends: <Minimum amount of healthy backends to be alive so the VIPs are created for that service>
        healthcheck: <HAproxy backend command>
        balance: <LB algorithm>
        frontend_port: <HAproxy frontend port for the service>
        haproxy_monitor_port: <HAproxy monitoring port for the service>
        backend_opts: <HAproxy options for each backend>
        backend_port: <HAproxy backend port>
        backend_hosts:
          - name: <hostname of the backend>
            ip: <IP of the backend>
```

Have a look at `tests/vars.yml` for a complete example.

Dependencies
------------

* `ansible.posix`

Example Playbook
----------------

Create a file named `playbook.yml`:
```
---
- hosts: lb1
  vars:
    config: lb1
  tasks:
    - name: Include vars for testing
      ansible.builtin.include_vars: vars.yml
    - name: Run the role
      include_role:
        name: emilienm.routed_lb
```

And then create a file named `inventory` for the Ansible inventory:
```
all:
  hosts:
    lb1:
      ansible_host: 192.168.10.2 # IP address of your Load-Balancer
      ansible_user: cloud-user
      ansible_become: true
```

Then run:
```
ansible-playbook playbook.yml -i inventory
```

Your load-balancers should be up and running!

License
-------

Apache-2.0
