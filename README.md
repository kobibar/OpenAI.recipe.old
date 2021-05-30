# UFM/SM/SHARP configuration in OpenAI
Recommended configuration for large-scale setup - OpenAI

- [OpenAI recipe](OpenAI.recipe.md)
- [Topology API](TopologyAPI.md)

# Notes

## ibdiagnet

Speedup ibdiagnet run:

- Redirect output to temporary file
- Increase SMP/GMP window size. SMP window = 256, can be used only if UFM machine was configured with paramters
```
ib hca-smp-window 128
ib hca-vl15-window 32
```

Example:
```bash
$ibdiagnet ... --gmp_window 8192 --smp_window 256> /tmp/ibdiag.output
```

## Location of debug dumps

ftp: ftp://ftpsupport.mellanox.com
folder: SHARP-Debug

For access credential, please contact NVIDA technical support

