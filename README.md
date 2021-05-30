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
## Location of debug dumps

ftp: ftp://ftpsupport.mellanox.com
folder: SHARP-Debug

For access credential, please contact NVIDA technical support

