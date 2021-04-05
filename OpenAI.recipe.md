# OpenAI Configuration

## Changelog
| Date              | Description                                             |
|-------------------|---------------------------------------------------------|
| 22 March, 2021    | Initial version                                         |
| 05 April, 2021    | Added "Rail-Optimized" and "SHARP" validations          |

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

## TBD & TODOs
1. Name of ibdiagnet in UFM Appliance: ib_ibdiagnet ???? (Kobi)
