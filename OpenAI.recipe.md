# OpenAI Configuration

Date: 22 March, 2021

## UFM Appliance

Changes in SM and SHARP configuration files need export/import from/to UFM Appliance
See: ["Modifying UFM Configuration Files" in UFM Appliance documentation](https://docs.mellanox.com/display/UFMSDNAppUMv450/Modifying+UFM+Configuration+Files) 

### Phase 1 (Performance tuning for large IB subnet)

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


### Phase 2 (SHARP)

1. Enable SHARP

```
ib sharp enable
ib sm sharp enable
ib sharp smx-protocol ucx
```

## SM

### SM settings
Modify: `conf/opensm/opensm.conf`

```
#Amount of physical port to handle in one shot
virt_max_ports_in_process 512

max_operational_vls 2

# Double check
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

# 0 - defsult (use all possible threads)
smp_threads 0
gmp_threads 0

# number of SMP MADs per device
max_wire_smps_per_device 2

enable_inc_mc_routing TRUE
```

### PGRP policy
Modify pgrp_policy_file: `/opt/ufm/conf/opensm/pgrp_policy.conf` file

```
port-group
port-regexp: dsm.*ib4
name: root_switches
end-port-group
```

### Root guids

Write into root_guid_file: `/opt/ufm/files/conf/opensm/root_guid.conf` name of policy

```
root_switches
```

### TBD & TODOs
1. pgpl and root guids file must be: import/export (Kobi)
## Tools (ibdiagnet)

### AR (Adaptive Routing) validation

For AR validation run:

```
ibdiagnet --routing --r_opt=dump_only --skip lids,sm,nodes_info,pkey,aguid,links,pm,speed_width_check,temp_sensing,virt
```

Output:

```
Summary
-I- Stage                     Warnings   Errors     Comment
-I- Discovery                 0          0
-I- Routing                   0          0
```

AR configuration dump is in : `/var/tmp/ibdiagnet2/ibdiagnet2.far` file.

The command above only dumps AR configuration. For actual checking run:

```
ibdiagnet --routing --skip lids,sm,nodes_info,pkey,aguid,links,pm,speed_width_check,temp_sensing,virt
```

### TBD & TODOs

1. Name of ibdiagnet in UFM Appliance: ib_ibdiagnet ???? (Kobi)
2. Rail optimized validation
3. Number of switches with AR configured
4. Community post to AR (Vladimir)

## SHARP

Modify `conf/sharp2/sharp_am.cfg`

1. Timeout for handling initial IB fabric configuration in SM
2. Recommended tree radix (will be out-of-box in future versions)
3. SL for SHARP packets

```
ib_qpc_sl 1
ib_sharp_sl 1
fabric_update_interval 10
lst_file_timeout 10
lst_file_retries 30
tree_radix 40
```

### TBD and TODOs

1. Radix : 40 vs 80
2. SAT SL Configuration : SAT vs LLT
3. SHARP section should include all steps

