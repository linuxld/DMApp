# NIM - Network Instalation Manager
Author: Marcelo F. Oliveira ( moliv@kyndryl.com )

### Automation Indice
  - [Infrastructure Creation](#Infrastructure-Creation)
  - [Software Download](#Software-Download)
  - [Configure NIM Server](#Configure-NIM-Server)


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
You will need point the Product, Version and Function. Ideally is to download it direct on the NIM Server when never it's possible. If doing so, you can create a folder under the LPP Source Filesystem using the ML name, like `7200-05-05-2246` and so on.
### Recommended to create as following:
- `mkdir -p /export/lpp_source/[ML Name]`

## Mount ISO as looping
loopmount -i [ISO Full Path] -m [Mount Point] -o "-V cdrfs -o ro"

### Recommended to create as following:
- `loopmount -i [ISO Full Path] -m /mnt -o "-V cdrfs -o ro"`

## Create installed software package
bffcreate -v -d [BFF Source] -t [BFF Target] [Package Name]

### Recommended to create as following:
- `bffcreate -v -d /mnt/installp/ppc -t /export/lpp_source/[Version] ALL`

## Umount the ISO
- `umount /mnt`

# Configure NIM Server


