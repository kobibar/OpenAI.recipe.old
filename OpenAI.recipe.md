# OpenAI Configuration

## Changelog
| Date              | Description                                             |
|-------------------|---------------------------------------------------------|
| 22 March, 2021    | Initial version                                         |
| 05 April, 2021    | Added "Rail-Optimized" and "SHARP" validations          |
| 29 April, 2021    | Added UFM Settings                                      |

## References

- ["Modifying UFM Configuration Files" in UFM Appliance documentation](https://docs.mellanox.com/display/UFMSDNAppUMv450/Modifying+UFM+Configuration+Files) 
- [How To Configure Adaptive Routing and SHIELD (New)](https://community.mellanox.com/s/article/How-To-Configure-Adaptive-Routing-and-SHIELD-New)


## UFM Appliance
Changes in SM and SHARP configuration files need export/import from/to UFM Appliance

### Performance tuning for large IB subnet
1. Connect UFM Appliance by two ports to two different leaf switches
2. Enable SM multi-port configuration in UFM Appliance
3. VL15 buffer `hca-vl15-window` should be configured to 32KB (max 128KB)
4. SMP Window configuration
	1. Kernel: 128
	2. SM: 256
	3. ibdiagnet: 256
5. GMP Window configuration: 8192

```
ufm multi-port-sm enable
ib sm smp-window 256
ib ibdiagnet smp-window 256
ib hca-smp-window 128
ib hca-vl15-window 32
ib ibdiagnet gmp-window 8192
```

`hca-vl15-window` and `hca-smp-window` required reboot of UFM Appliance.


## UFM

### UFM settings
Modify: `conf/gv.cfg`

```
[Server]
# report_events that will determine which trap to send  to ufm all/security/none
report_events = security

# This parameter defines the polling frequency of default session in seconds, from 10 to 60
dashboard_interval = 300

# minimal possible interval for monitoring session  should be exposed to GUI (under site). GUI will not allow put value that lower than this.
# Server will check monitoring session and replace lower values by this.(in seconds, from 1 to 10)
minimal_collection_interval = 300

# default interval for monitoring session  should be exposed to GUI (under site).  The value will appear in Monitoring session dialog.
#  (in seconds, from 3 to 15)
default_collection_interval = 300

# interval for checking fabric non-optimal links
non_opt_links_check_interval = 10800

# time interval that port did not get accounting info and should be reset
ibpm_counters_reset_interval = 3000

# set optional Site name to be shown in event sent to syslog
site_name = <set meaningful site-name>


[FabricAnalysis]
# unmanaged_switches_interval (in minutes) - time interval between 2 sequential runs of fabric analysis for unmanaged switches
unmanaged_switches_interval = -1

# ibdiagnet periodic run interval for cables discovery - runs only if a links were added to the fabric(in minutes)
periodic_discovery_interval = 30

# timeout for ibdiagnet run time (in seconds)
ibdiagnet_timeout = 600


[Sharp]
# This parameter defines if sharp process will be running, or not
# default is false, no need to run sharp aggregation manager
sharp_enabled = true


[SrvMgmt]
# Interval for switches inventory discovery (in seconds)
systems_poll = 10800

# delay between 2 sequential sysinfo calls of 2 systems.(in seconds)
events_time_interval = 300


[IBPM]
comp_ib_slow_interval = 300

# by default, UFM is not setting the UDP buffer size, for large scale fabrics, it
# is recommended to increase the buffer size to 4M
set_udp_buffer = yes

# UDP buffer size
udp_buffer_size = 4194304


[UFMAgent]
# Whether to enable/disable the UFM Agent discovery
enable_ufma = no


[Events]
# Whether to suppress (disable) link down events in case switch went down
suppress_link_down_events_upon_switch_down = true

# Max events list size kept in UFM model
max_ufm_events = 1000


[DailyReport]
# Whether to run daily report or not
daily_report_enabled = false


[UnhealthyPorts]
# Enable/disable ibdiagnet run for all fabric for ports LLR detection
enable_ibdiagnet = false
```

## SM

### SM settings
Modify: `conf/opensm/opensm.conf`

```
#Amount of physical port to handle in one shot
virt_max_ports_in_process 512

max_operational_vls 2

qos TRUE

# Single MAD Sl2vl for all ports
use_optimized_slvl TRUE 

# Timeout for long MAD config time. might need to change 1000 
long_transaction_timeout 500 

routing_engine ar_updn

use_ucast_cache TRUE

root_guid_file /opt/ufm/files/conf/opensm/root_guid.conf
pgrp_policy_file /opt/ufm/files/conf/opensm/pgrp_policy.conf

scatter_ports 8
# 0 - maximum
routing_threads_num 24

# 0 - default (use all possible threads)
smp_threads 0
gmp_threads 0

# number of SMP MADs per device
max_wire_smps_per_device 2

enable_inc_mc_routing TRUE
```

### PGRP policy
Modify pgrp_policy_file: `conf/opensm/pgrp_policy.conf` file using export/import procedure.

```
port-group
port-regexp: dsm.*ib4
name: root_switches
end-port-group
```

### Root guids
Add root guid's policy name to root_guid_file: `conf/opensm/root_guid.conf` using export/import procedure.

```
root_switches
```

## Tools (ibdiagnet)

### AR (Adaptive Routing) validation
For AR validation run:

```
ibdiagnet --routing --r_opt=dump_only --skip lids,sm,nodes_info,pkey,aguid,links,pm,speed_width_check,temp_sensing,virt
```

Output:

```
---------------------------------------------
Fabric Summary

Total Nodes             : 15
IB Switches             : 3
IB Channel Adapters     : 9
IB Aggregation Nodes    : 3
IB Routers              : 0

Adaptive Routing is enabled on 3 switches

---------------------------------------------
Summary
-I- Stage                     Warnings   Errors     Comment
-I- Discovery                 3          0
-I- Virtualization            0          0
-I- Routing                   0          0
```

AR configuration dump is in : `/var/tmp/ibdiagnet2/ibdiagnet2.far` file.

The command above only dumps AR configuration. For actual checking run:

```
ibdiagnet --routing --skip lids,sm,nodes_info,pkey,aguid,links,pm,speed_width_check,temp_sensing,virt
```

### "Rail-Optimized" connectivity validation
OpenAI is "rail-optimized". That means, compute nodes are multi-port and each leaf switch connects IB ports in compute nodes located in the same position.
For checking connectivity run: 

```
ibdiagnet --skip lids,sm,nodes_info,pkey,aguid,links,pm,speed_width_check,temp_sensing,virt --rail_validation
```

Output:
```
---------------------------------------------
Rail Optimized Topology Validation
-I- Rail Optimized Topology Validation finished successfully

---------------------------------------------
```

## SHARP
SHARP should be enabled only in "stable" IB network.

### UFM Appliance - CLI 
```
ib sharp enable
ib sm sharp enable
ib sharp smx-protocol ucx
ib sharp topology-api enable
```

### UFM Appliance - SHARP configuration file

Modify `conf/sharp2/sharp_am.cfg`

```
ib_qpc_sl 1
fabric_update_interval 10
lst_file_timeout 10
lst_file_retries 30
tree_radix 80
```

### SHARP Validation
For SHARP validation run:

```
sudo ibdiagnet --sharp --skip lids,sm,nodes_info,pkey,aguid,links,pm,speed_width_check,temp_sensing,virt
```

The command above reports status of SHARP
```
-I- SHARP check trees finished successfully

---------------------------------------------
Fabric Summary

Total Nodes             : 24
IB Switches             : 4
IB Channel Adapters     : 16
IB Aggregation Nodes    : 4
IB Routers              : 0

Number of aggregation nodes version 1: 4
Number of aggregation nodes version 2: 0
Number of LL aggregation trees: 2
Number of streaming aggregation trees: 0
Unique job IDs: 1024
```
