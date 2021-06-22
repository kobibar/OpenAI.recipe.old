# UFM/SM/SHARP configuration in OpenAI
Recommended configuration for large-scale setup - OpenAI

- [OpenAI recipe](OpenAI.recipe.md)
- [Topology API](TopologyAPI.md)

# Notes

## ibdiagnet

Speedup ibdiagnet run:

- Redirect output to temporary file
- Increase SMP/GMP window size. SMP window = 256, can be used only if UFM machine was configured with parameters
- Skip fetching virtual ports, if this information is not required 
```
ib hca-smp-window 128
ib hca-vl15-window 32
```

Examples:
```bash
$ibdiagnet --skip aguid --gmp_window 8192 --smp_window 256 ..... > /tmp/ibdiag.output
```

```bash
$ibdiagnet --skip aguidi,virt --gmp_window 8192 --smp_window 256 --pm_pause_time 60 --routing --r_opt rn,drnc --extended_speeds all --counter all --pm_per_lane --get_cable_info --cable_info_disconnected --get_phy_info --get_p_info > /tmp/ibdiag.output
```

### How to find "socket-direct" HCAs

The command below prints node guids of HCAs reporting "socket-direct" mode (the data comes from pcie counters)

```bash
$ sudo ibdiagnet  --get_p_info
$ awk -F,  'BEGIN{s=0}; /START_P_DB2/{s=1} ; /END_P_DB2/{s=0} ; s==1 &&  $6=="1"  {print $1}' ibdiagnet2.db_csv | sort | uniq
```

## Location of debug dumps

ftp: ftp://ftpsupport.mellanox.com
folder: SHARP-Debug

For access credential, please contact NVIDA technical support

## How to enable GID based routing

1. Set the NVRAM
``` bash
# mlxconfig -d mlx5_<x> s SRIOV_IB_ROUTING_MODE_P1=0
```

2. Verify the setting
```bash
# mlxconfig -d mlx5_<x> -e q | grep -i -e config -e sriov_ib
Configurations:                              Default         Current         Next Boot
        SRIOV_IB_ROUTING_MODE_P1            LID(1)          GID(1)          GID(0)
```

3. Reset firmware or reboot the machine
```bash
# mlxfwreset -d mlx5_<x> -l 3 -y r
```

4. Verify the setting
```bash
# mlxconfig -d mlx5_<x> -e q | grep -i -e config -e sriov_ib
Configurations:                              Default         Current         Next Boot
        SRIOV_IB_ROUTING_MODE_P1            LID(1)          GID(0)          GID(0)
```

5. Restart AM
