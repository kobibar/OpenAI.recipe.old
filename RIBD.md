# RIBD (Remote IBDiagNet)

&copy; Copyright 2021, NVIDIA CORPORATION & AFFILIATES, Inc. All Rights Reserved.

See file LICENSE for terms.

July 2021

[TOC]



# Delivery

RIBD service delivered as two docker containers for server (UFM Appliance) and client (for HOST machine)

Notes for installation see in guide

# Installation Guide

All commands contain RIBD version v0.0.7. You should replace v0.0.7 to your version number.

## Server

### Enable docker service

``` bash
$ no docker shutdown
```

### Deploy the container

``` bash
$ docker pull <image name>
```
or (followed by the command �docker load ribd_server.v0.07.tgz�)
``` bash
$ image fetch scp://<username>@<host>:/hpc/noarch/RIBD/0.0.7/ribd_server.v0.0.7.tgz
```

### Show the image [Optional] 

``` bash
$ show docker images
```

### Create shared volumes between the appliance and the container

``` bash
$ docker label ribd.config
```

### Load Image

```bash
$ docker load ribd_server.v0.0.7.tgz
```

### Start the container

``` bash
$ docker start ribd_server 0.0.7 ribd_server now-and-init privileged network label ribd.config 
```

### Save the configuration changes

``` bash
$ configuration write
```

### Show the container is running [Optional] 

``` bash
$ show docker ps
```


## Client

### Download client package to the temporary folder 

``` bash
$ wget http://server/ribd_client.v0.0.7.tgz -P /tmp
```



### Load docker image 

``` bash
$ docker load < /tmp/ribd_client.v0.0.7.tgz
8f8f0266f834: Loading layer [==================================================>]  65.49MB/65.49MB
2776c26cfa81: Loading layer [==================================================>]  86.43MB/86.43MB
6265c8df1205: Loading layer [==================================================>]  1.231GB/1.231GB
79d9a76b0833: Loading layer [==================================================>]   2.56kB/2.56kB
902ef1c172b6: Loading layer [==================================================>]  1.466MB/1.466MB
Loaded image: ribd_client:0.0.7
```



# User Guide



## Execute ibdiagnet



1. Start docker 

```bash
$ docker run --rm -it -v <config_local_path>:/opt/ribd_client/etc/ribd/ --privileged ribd_client:0.0.7 /bin/bash
   
$ root@c9fcff21895e:/#
$
$ 
```

   Docker must be run with --privileged 

   You should mount external directory for store configuration. 

   Path for mount in docker '/opt/ribd_client/etc/ribd/'

   Default configuration will be created at the first run. 



2. Run ibdiagnet

```bash
   $  /opt/ribd_client/bin/ribd_client
```

   Result tar file will be stored in current folder

## Configuration 

In first start default configuration will be created default configuration 

```
#Device and port for open UCX connection
device          mlx5_0
port            1

#Connection mode DC enabled, by default used RC
dc_mode         0

#Timeouts section (not recommended to change)
wait_sleep      100
recv_timeout    5000
send_timeout    5000
flush_timeout   5000

#Service record section (not recommended to change)
srm_id          22
srm_name        RIBD
srm_retries     5

#KeepAlive timeout in seconds (not recommended to change)
keepalive       10
```

 Limitation : Config file doesn't support commentaries and will be not present in created configuration file at the first run.



## Command line parameters

Command line parameters of original ibdiagnet can be used in RIBD with few exclusions.

Part of original ibdiagnet command line parameters will be override for local usage for RIBD client. 

### Local RIBD clients command line parameters

| Parameter name  | Description                                                  |
| :-------------- | ------------------------------------------------------------ |
| `---v`          | Show RIBD client version                                     |
| `---vv`         | Show RIBD client & server version                            |
| `-i | --device` | Specifies the name of the device to connect to the IB fabric |
| `-p | --port`   | Specifies the port of the device to connect to the IB fabric |
|                 |                                                              |

### Exclusions 

Next command line parameters cannot be used with RIBD

| Parameter name                                         | Description                                                  |
| :----------------------------------------------------- | ------------------------------------------------------------ |
| `-o|--output_path <directory>`                         | Specifies the directory where the output files will be placed. |
| `-c|--create_config_file <config-file>`                | Create template configuration file.                          |
| `-g|--guid <GUID in hex>`                              | Specifies the local port GUID value of the port used to connect to the IB fabric. If GUID given is 0 then ibdiagnet displays a list of possible port GUIDs and waits for user input. |
| `--smp_window <num>`                                   | Max smp MADs on wire. (default=16).                          |
| `--gmp_window <num>`                                   | Max gmp MADs on wire. (default=256).                         |
| `-m|--map <map-file>`                                  | Specifies mapping file, that maps node guid to name (format: 0x[0-9a-fA-F]+ "name"). Mapping file can also be specified by environment variable "IBUTILS_NODE_NAME_MAP_FILE_PATH". |
| `--m_key_files <path to m_key_files directory>`        | Specifies the path to the directory with the mkey_files (guid2lid, guid2mkey, neighbors). |
| `--config_file <config-file>`                          | Configuration file                                           |
| ` --sa_dump <file>`                                    | Specifies opensm-sa.dump file path, multicast groups definition generated by SM. used for mcast credit loop check (if -r option selected and r_opt=mcast). |
| `-w|--write_topo_file <file name>`                     | Write out a topology file for the discovered topology.       |
| `-t|--topo_file <file>`                                | Specifies the topology file name.                            |
| `--path <files types list>=<path>`                     | Set custom path for specific files                           |
| `--exclude_scope <file.guid>`                          | The file with a list of Node-GUIDs and their ports to be excluded from the scope. |
| `--vlr <file>`                                         | Specifies opensm-path-records.dump file path, src-dst to SL mapping generated by SM plugin. ibdiagnet will use this mapping for MADs sending and credit loop check (if -r option selected). |
| `-f|--load_from_file <path to ibdiagnet2.db_csv file>` | Load ibdiagnet from file. Use this option to skip discovery stage. The input file must be a valid output of ibdiagnet. |
| `--write_capability <file name>`                       | Write out an example file for capability masks configuration, and also the default capability masks for some devices. |
|                                                        |                                                              |
|                                                        |                                                              |
|                                                        |                                                              |
|                                                        |                                                              |
|                                                        |                                                              |
|                                                        |                                                              |

### Special exclusion

RIBD cannot be used for routing validation.

Command line parameter '-r' or '--routing' can be used only with '--r_opt=dump_only'

## Examples

### Start docker

```bash
docker run -it  -v /tmp:/opt/ribd_client/etc/ribd/ --privileged ribd_client:0.0.8 /bin/bash
```

* Don't forget '--privileged'
* Don't forget mount local folder for store configuration. On start first time will be created default configuration file, possible need config device and port for connection to the fabric. Second solution without prepared/edited configuration file you can use '-i' & '-p' for config via command line.

 ### Docker inside

```
root@4b06f83efa00:/# /opt/ribd_client/bin/ribd_client
------------------------------------------------------
The provide configuration file: /opt/ribd_client/etc/ribd/ribd_client.conf
The configuration file was read successfully
Start logger, file '/tmp/ribd/ribd_client.log'
Run remote 'ibdiagnet '
Running version: "IBDIAGNET 2.6.2.MLNX20210321.84ba964","IBDIAG 2.1.1.84ba964","IBDM 2.1.1.84ba964","IBIS 2.1.1.386adda"
Running command: ibdiagnet -i mlx5_4 -p 1 --path all=/tmp/ribd/ribd_456700bdfb70/
Timestamp: 2021-07-14 07:14:59 UTC +0000

..
.. OUTPUT
..

---------------------------------------------
Summary
-I- Stage                               Warnings   Errors     Comment
-I- Discovery                           0          0
-I- Lids Check                          0          0
-I- Links Check                         0          0
-I- Subnet Manager                      0          0
-I- Port Counters                       0          0
-I- Nodes Information                   2          0
-I- Speed / Width checks                0          13
-I- Alias GUIDs                         0          0
-I- Virtualization                      0          1
-I- Partition Keys                      0          0
-I- Temperature Sensing                 0          0
-I- Create IBNetDiscover File           0          0

-I- You can find detailed errors/warnings in: /tmp/ribd/ribd_456700bdfb70/ibdiagnet2.log


-I- Database                            : /tmp/ribd/ribd_456700bdfb70/ibdiagnet2.db_csv
-I- LST                                 : /tmp/ribd/ribd_456700bdfb70/ibdiagnet2.lst
-I- Network dump                        : /tmp/ribd/ribd_456700bdfb70/ibdiagnet2.net_dump
-I- Subnet Manager                      : /tmp/ribd/ribd_456700bdfb70/ibdiagnet2.sm
-I- Ports Counters                      : /tmp/ribd/ribd_456700bdfb70/ibdiagnet2.pm
-I- Nodes Information                   : /tmp/ribd/ribd_456700bdfb70/ibdiagnet2.nodes_info
-I- Alias guids                         : /tmp/ribd/ribd_456700bdfb70/ibdiagnet2.aguid
-I- VPorts                              : /tmp/ribd/ribd_456700bdfb70/ibdiagnet2.vports
-I- VPorts Pkey                         : /tmp/ribd/ribd_456700bdfb70/ibdiagnet2.vports_pkey
-I- Partition keys                      : /tmp/ribd/ribd_456700bdfb70/ibdiagnet2.pkey
-I- IBNetDiscover                       : /tmp/ribd/ribd_456700bdfb70/ibdiagnet2.ibnetdiscover

---------------------------------------------
Prepare result archive...
ribd_456700bdfb70/
ribd_456700bdfb70/ibdiagnet2.pkey
ribd_456700bdfb70/ibdiagnet2.ibnetdiscover
ribd_456700bdfb70/ibdiagnet2.db_csv
ribd_456700bdfb70/ibdiagnet2.vports
ribd_456700bdfb70/ibdiagnet2.nodes_info
ribd_456700bdfb70/ibdiagnet2.debug
ribd_456700bdfb70/ibdiagnet2.sm
ribd_456700bdfb70/ibdiagnet2.aguid
ribd_456700bdfb70/ibdiagnet2.net_dump
ribd_456700bdfb70/ibdiagnet2.log
ribd_456700bdfb70/ibdiagnet2.output
ribd_456700bdfb70/ibdiagnet2.pm
ribd_456700bdfb70/ibdiagnet2.lst
ribd_456700bdfb70/ibdiagnet2.vports_pkey

---------------------------------------------
Transfer result archive...
```

Result will be stored in current folder './ribd_456700bdfb70.tar,gz'

### Using scope file

```
root@491690d14156:/# /opt/ribd_client/bin/ribd_client --scope example.guid
------------------------------------------------------
The provide configuration file: /opt/ribd_client/etc/ribd/ribd_client.conf
The configuration file was read successfully
Start logger, file '/tmp/ribd/ribd_client.log'
Run remote 'ibdiagnet '
```

You can provide local 'scope' file or other original ibdiagnet options directly without modification.

For exclusion see table 'Exclusions'

