# NIM - Network Instalation Manager
Author: Marcelo F. Oliveira ( moliv@kyndryl.com )

### Indice
  - [Infrastructure Creation](#Infrastructure-Creation)
  - [Software Download](#Software-Download)
  - [Configure NIM Server](#Configure-NIM-Server)
  - [Manage NIM Resouces](#Manage-NIM-Resouces)
  - [Manage NIM Client](#Manage-NIM-Client)

# Infrastructure Creation

## Create Volume Group
mkvg -S -s [Block Size] -y [VG Name] -f [Disk Name]

### Recommended to create as following:
- `mkvg -S -s 64 -y vg_nim -f hdisk1`

## Create Logical Volumes 
mklv -t jfs2 -y [LV Name] [VG Name] -L [Size in MB]

### Recommended to create as following:
- `mklv -t jfs2 -y lv_nimspot vg_nim -L 4096`
- `mklv -t jfs2 -y lv_nimlpp vg_nim -L 8192`
- `mklv -t jfs2 -y lv_nimrepo vg_nim -L 4096`
- `mklv -t jfs2 -y lv_nimimg vg_nim -L 8192`

## Create Filesystems
crfs -d [LV Name] -v jfs2 -A yes -a logname=INLINE -m [Mount Point]

### Recommended to create as following:
- `crfs -d lv_nimspot -v jfs2 -A yes -a logname=INLINE -m /export/spot`
- `crfs -d lv_nimlpp -v jfs2 -A yes -a logname=INLINE -m /export/lpp_source`
- `crfs -d lv_nimrepo -v jfs2 -A yes -a logname=INLINE -m /export/repository`
- `crfs -d lv_nimimg -v jfs2 -A yes -a logname=INLINE -m /export/mksysb`

## Create and mount Filesystems
mkdir -p [Mount Point]; mount [Mount Point]

### Recommended to create as following:
- `mkdir -p /export/spot; mount /export/spot`
- `mkdir -p /export/lpp_source; mount /export/lpp_source`
- `mkdir -p /export/repository; mount /export/repository`
- `mkdir -p /export/mksysb; mount /export/mksysb`

## Export Repository Filesystems
mknfsexp -d [Mount Point] -v [NFS Version] -B -S 'sys,none' -t 'rm' -h '[NIM Server]' -c '-' -r '-,[NIM Server]'

### Recommended to create as following:
- `mknfsexp -d /export/repository -v 3 -B -S 'sys,none' -t 'rm' -h '$(hostname)' -c '-' -r '-,$(hostname)'`

# Software Download 
## ISOs
Need to post the location where to download the ISOs

## ML and Fixes
It can be downloaded on IBM Central Fix https://www.ibm.com/support/fixcentral website.
You will need point the Product, Version and Function. Ideally is to download it direct on the NIM Server when never it's possible. If doing so, you can create a folder under the Repository Filesystem would be simple to a folder as ML name is present on IBM Central Fix, like 7200-05-05-2246.
### Recommended to create as following:
- `mkdir -p /export/repository/7200-05-05-2246`

## Mount ISO as looping
loopmount -i [ISO Full Path] -m [Mount Point] -o "-V cdrfs -o ro"

### Recommended to create as following:
- `loopmount -i /export/repository/aix72_cd01.iso  -m /mnt -o "-V cdrfs -o ro"`

## Create installed software package
bffcreate -v -d [BFF Source] -t [BFF Target] [Package Name]

### Recommended to create as following:
- `bffcreate -v -d /mnt/installp/ppc -t /export/repository/aix72cd ALL`

## Umount the ISO
- `umount /mnt`

# Configure NIM Server

## Define NIM master
nimconfig -a pif_name=[NIC] -a cable_type=tp -a comments=["Commentary"] -a net_addr=[Network Address] -a snm=[Network Mask] -a routing1="default [Network Gateway]" [Network Name]

### Recommended to create as following:
- `nimconfig -a pif_name=en0 -a cable_type=tp -a comments="Network on vLAN100" -a net_addr=192.168.0.0 -a snm=255.255.255.0 -a routing1="default 192.168.0.1" vlan100_net`
- NIC: It can be obtained by the command `netstat -i` or `ifconfig -a`
- Network Name: Is recomended to use the vLAN as a name, it t can be obtained by the command `entstat -d [NIC]|grep -i vlan`
- Network Address: It can be obtained by the command `lsattr -El inet0`
- Network Mask: It can be obtained by the command `lsattr -El inet0`
- Network Gateway: It can be obtained by the command `lsattr -El inet0`

## Define LPP Source
nim -o define -t lpp_source -a server=master -a source=[BFF Source] -a location=[LPP Source Filesystem]/[LPP Name] -a comments=["Commentary"] [LPP Name]

### Recommended to create as following:
- `nim -o define -t lpp_source -a server=master -a source=/export/repository/aix72cd -a location=/export/lpp_source/aix72_lpp -a comments="LPP Source for AIX 7.2" aix72_lpp`

## Update LPP Source from the ML
nim -o update -a location=[ML Location] [LPP Source Name]

### Recommended to create as following:
- `nim -o update -a location=/export/repository/7200-05-05-2246 aix72_lpp`

## Purge the duplicate and superseded packages
nim -o lppmgr -a lppmgr_flags="-rbux" [LPP Source Name]

### Recommended to create as following:
- `nim -o lppmgr -a lppmgr_flags="-rbux" aix72_lpp`

## Purge the all languages packages except en_US
nim -o lppmgr -a lppmgr_flags="-r -k en_US" [LPP Source Name]

### Recommended to create as following:
- `nim -o lppmgr -a lppmgr_flags="-r -k en_US" aix72_lpp`

## Define Spot
nim -o define -t spot -a server=master -a source=[LPP Name] -a location=[Spot Filesystem]/[Spot Name] -a comments=["Commentary"] [Spot Name]

### Recommended to create as following:
- `nim -o define -t spot -a server=master -a source=aix72_lpp -a location=/export/spot/aix72_spot -a comments="Spot for AIX 7.2" aix72_spot`

## Clean up all the unnecessary files
- `rm -rf /export/repository/7200-05-05-2246`
- `rm -rf /export/repository/aix72cd`

# Manage NIM Resouces

## Destroy NIM Resource
nim -Fo destroy [Resource]

### Recommended to create as following:
- Resource: It can be obtained by the command `lsnim`

## Define NIM Resource
- Define machine: `nim -o define -t machine -a name=[Hostname] -a platform=chrp -a if1=[Network Name] -a cable_type1=tp`
- Define mksysb: `nim -o define -t mksysb -a server=master -a location=/export/mksysb[Image Name] -a source=[Machine Name] -a mk_image=yes [Resource Name]`

## Deallocate NIM Resource
- LPP Source: `nim -o deallocate -a lpp_source=[LPP Source Name] [Machine Name]`

# Manage NIM Client

## AIX 
- Install: `nim -o cust -a filesets=[Fileset Name] -a lpp_source=[LPP Name] -a targets=[Target List] -a accept_licenses=yes`
- Update: `nim_update_all -l [LPP Source Name] -s [Spot Name] -B [Machine]`
- Commit: `nim -o updateios -a updateios_flags=-commit [VIOS Machine]`

## VIOS 
- Update: `nim -o updateios -a lpp_source=[LPP Name] -a preview=no [VIOS Machine]`
- Commit: `nim -o updateios -a updateios_flags=-commit [VIOS Machine]`
