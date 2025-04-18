# exasol_migration

# Exasol deployment. 
##### NOTE: Full doc under https://docs.exasol.com/db/latest/administration/aws/installation.htm 
 - You need to configure the aws cli and aws credentials (use service_isc_pdap_cloudetl02 aws profile), and an existent security group for the 
previous version 7 clusters, and add port 20003 and 20002 (for more info look at the AWS isc-bi-stg01 account into sg for dev sg-0e578844cc94f6f60 - isc-imt-exasol-dev-DBSecurityGroup-1SWOHESN7XLLE for the needed inbound rules)
##### NOTE: Please also check the instance type and the data voulumes size that need to be passed to the c4 config.
 - Then install the c4 too using (version = 4.24.3 for exasol 8.33.0 latest version) and later add it to the PATH: 
```bash
wget https://x-up.s3.amazonaws.com/releases/c4/linux/x86_64/2.24.3/c4 -O c4 && chmod +x c4
```
 - Then create a ~/.ccc folder and a file ~/.ccc/config as this example:
```bash
CC_USER_EMAIL=devops@eventim.de
CCC_PLAY_ACCESS_NODE=true
CCC_PLAY_ADMIN_PASSWORD=PyxpyM-Fubxyz-Xycfu0
CCC_PLAY_DB_PASSWORD=Dyxpym-Bubxyz-Aycfu0
CCC_USER_PASSWORD=Byxpym-fubxyz-Cycfu0
CCC_PLAY_DB_PORT=8563
CCC_PLAY_DATABASE_NAME=isc_dev_cloud_dwh
CCC_AWS_USE_EIP=false
CCC_AWS_IMAGE_ID=ami-02daf032f53622cf6
CCC_AWS_INSTANCE_TYPE=c5ad.4xlarge
CCC_AWS_NO_MFA=true
CCC_AWS_PROFILE=service_isc_pdap_cloudetl02
CCC_AWS_SECURITY_GROUP_ID=sg-0e578844cc94f6f60
CCC_AWS_SECURITY_GROUP_NAME=isc-imt-exasol-dev-DBSecurityGroup-1SWOHESN7XLLE
CCC_AWS_REGION=eu-central-1
CCC_AWS_KEY_PAIR=ISC_IMT
CCC_AWS_KEY_PAIR_FILE=ISC_IMT.pem
CCC_AWS_NEW_VPC=false
CCC_AWS_SUBNET_ID=subnet-072997124ba8f352f
CCC_AWS_ROOT_DEVICE_SIZE=200
CCC_PLAY_OS_DISK_SIZE=120
CCC_PLAY_DATA_DISK_SIZE=690
CCC_PLAY_INSTANCES_IP_ADDRS=10.190.238.120,10.190.238.121,10.190.238.122
CCC_PLAY_NETWORK_CIDR=10.190.232.0/21
CCC_USER_PS_REMOTE_TIMEOUT=10000
CCC_AWS_DISABLE_METADATA_SERVICE=false
CCC_UPDATE_TEMPLATE_FORCE=false
CCC_PLAY_NAME=isc-imt-exasol8-dev
```
-- Before doing the deployment do c4 aws diag to check that the needed tools are installed:
```bash
dw_etl@isc-pdap-cloudetl11:~> c4 aws diag
OK aws tools are installed
OK aws version 1.16+
OK aws tools credentials are set
OK aws tools credentials are correct
OK exasol aws account is accessible
OK Private AWS SSH access key file found
OK CCC_USER_EMAIL set to 'devops@eventim.de'
dw_etl@isc-pdap-cloudetl11:~> 
```
-- And also do c4 config --validate :
```bash
dw_etl@isc-pdap-cloudetl11:~> c4 config --validate
CCC_AWS_DISABLE_METADATA_SERVICE=false
CCC_AWS_IMAGE_ID=ami-02daf032f53622cf6
CCC_AWS_INSTANCE_TYPE=c5ad.4xlarge
CCC_AWS_KEY_PAIR=ISC_IMT
CCC_AWS_KEY_PAIR_FILE=ISC_IMT.pem
CCC_AWS_NEW_VPC=false
CCC_AWS_NO_MFA=true
CCC_AWS_PROFILE=service_isc_pdap_cloudetl02
CCC_AWS_REGION=eu-central-1
CCC_AWS_ROOT_DEVICE_SIZE=200
CCC_AWS_SECURITY_GROUP_ID=sg-0e578844cc94f6f60
CCC_AWS_SECURITY_GROUP_NAME=isc-imt-exasol-dev-DBSecurityGroup-1SWOHESN7XLLE
CCC_AWS_SUBNET_ID=subnet-072997124ba8f352f
CCC_AWS_USE_EIP=false
CCC_PLAY_ACCESS_NODE=true
CCC_PLAY_ADMIN_PASSWORD=PyxpyM-Fubxyz-Xycfu0
CCC_PLAY_DATABASE_NAME=isc_dev_cloud_dwh
CCC_PLAY_DATA_DISK_SIZE=690
CCC_PLAY_DB_PASSWORD=Dyxpym-Bubxyz-Aycfu0
CCC_PLAY_DB_PORT=8563
CCC_PLAY_INSTANCES_IP_ADDRS=10.190.238.120,10.190.238.121,10.190.238.122
CCC_PLAY_NETWORK_CIDR=10.190.232.0/21
CCC_PLAY_OS_DISK_SIZE=120
CCC_UPDATE_TEMPLATE_FORCE=false
CCC_USER_EMAIL=devops@eventim.de
CCC_USER_PASSWORD=Byxpym-fubxyz-Cycfu0
CCC_USER_PS_REMOTE_TIMEOUT=10000
dw_etl@isc-pdap-cloudetl11:~> 
```
-- For the deployment to be success full you will need to add the following VPC endpoints using the security group, subnet (data subnet) and vpn from the config:
  - SSM Messages
  - SSM
  - CloudFormation
  - EC2
  - S3
  - KMS

-- If those endpoints are already configured then just add the security group and subnet used in the config to it.
-- Then for doing a normal (10 or less nodes) deployment do:
```bash
c4 aws play -N 2 -T @exasol-8.33.0
```
-- And check the deployment doing c4 ps or in CloudFormation after a couple of minutes (10-15 min):
##### NOTE: For this comment out the play name in the config (#CCC_PLAY_NAME=isc-imt-exasol8-dev) or c4 will not find any deployments
```bash
dw_etl@isc-pdap-cloudetl11:~> c4 ps
      N  PLAY_ID   NODE  MEDIUM  INSTANCE      DB_VERSION  EXTERNAL_IP  INTERNAL_IP    STAGE  STATE    UPTIME    TTL  
  ┌─  1  1e74b46c  10    awscf   c5ad.large    -           -            10.190.238.58  a      running  00:00:00  +∞   
  │   1  1e74b46c  11    awscf   c5ad.4xlarge  -           -            10.190.238.40  a      running  00:00:00  +∞   
  └─  1  1e74b46c  12    awscf   c5ad.4xlarge  -           -            10.190.238.25  a      running  00:00:00  +∞   
```
-- For the deplyment to be finish the access node (10) should be in stage c and the data nodes (11,12) in stage d fully deployed this
should take 5-10 minutes:
```bash
dw_etl@isc-pdap-cloudetl11:~> c4 ps
      N  PLAY_ID   NODE  MEDIUM  INSTANCE      DB_VERSION  EXTERNAL_IP  INTERNAL_IP    STAGE  STATE    UPTIME    TTL  
  ┌─  1  1e74b46c  10    awscf   c5ad.large    -           -            10.190.238.58  c      running  00:10:00  +∞   
  │   1  1e74b46c  11    awscf   c5ad.4xlarge  -           -            10.190.238.40  d      running  00:10:00  +∞   
  └─  1  1e74b46c  12    awscf   c5ad.4xlarge  -           -            10.190.238.25  d      running  00:10:00  +∞   
```
-- Now you can access the database using the CCC_PLAY_DB_PASSWORD and default user sys with the 
connection string (10.190.238.25,10.190.238.40) being coma separated list of the data nodes ips.
-- For a deplyment of more than 10 nodes use a custom template which can be anyone of the previously installed CouldFormation templates
just save then in a AWS s3 bucket:
```bash
CCC_AWS_TEMPLATE_URL='s3://x-u/ib4290/custom-tmpl' c4 aws play -N 12 -T @exasol-8.33.0
```

# Exasol migration step by step from exasol 7.1 to exasol 8 
##### NOTE: You need a cluster with the same number of data nodes and one access node in exasol 8 to migrate from exasol 7.1
##### IMPORTANT: The full documentation of this is on https://docs.exasol.com/db/latest/administration/aws/upgrade/migrate_71_v8.htm

1. Upload licence and create remote volume with access to the backups for exasol 7.1 version:
- First spet connect to the access node, upload the licence creating a file and copy pasting the content of the license to that file inside the access node
- Then create a remote volume linked to the s3 where the verison 7.1 backups are
NOTE: Add the s3 bucket arn into the instance role (isc-imt-exasol8-dev-InstanceRole) in orther to be accesible from our new exasol8 deployment, dont forget to add * to get access to the full bucket. 

```bash
c4 connect -t 1.10/cos
root@n10:~# confd_client license_upload license: '"{< ./2212000217.exasol_license}"'
Contract:
  comment: 'TEST AWS #22070--ServiceTag: 57A9FFA'
  company_name: CTS EVENTIM AG & Co. KGaA
  distributor: Exasol
  distributor_id: 1
  expiration_date: '2025-12-31'
  license_id: 2212000217
Exasol_DB_license:
  schema_version: 1
Limits:
  max_db_mem_size_in_gb: Unlimited
  max_db_raw_data_size_in_gb: Unlimited
  max_nodes_per_cluster: Unlimited
  max_num_clusters: Unlimited
root@n10:~# confd_client license_info 
Contract:
  comment: 'TEST AWS #22070--ServiceTag: 57A9FFA'
  company_name: CTS EVENTIM AG & Co. KGaA
  distributor: Exasol
  distributor_id: 1
  expiration_date: '2025-12-31'
  license_id: 2212000217
Exasol_DB_license:
  schema_version: 1
Limits:
  max_db_mem_size_in_gb: Unlimited
  max_db_raw_data_size_in_gb: Unlimited
  max_nodes_per_cluster: Unlimited
  max_num_clusters: Unlimited
root@n10:~# confd_client remote_volume_add url: https://isc-imt-exasol-dev-s3bucket-jfloi6li7zzm.s3.eu-central-1.amazonaws.com/ vol_type: s3 remote_volume_name: r0000 owner: [500,500]
root@n10:~# confd_client remote_volume_info remote_volume_name: r0000
name: r0000
owner:
- 500
- 500
type: s3
url: https://isc-imt-exasol-dev-s3bucket-jfloi6li7zzm.s3.eu-central-1.amazonaws.com/
vid: '10003'
```

2.- Backup and restore from (level 0 + level 1)
 -- After creating a remote volume pointing into the exasol 7 bucket with the backups list the backups and pick the more recent level 1 backup
```bash
dw_etl@isc-pdap-cloudetl11:~/GIT/py_database_tools> c4 connect -t 1.10/cos
Last login: Thu Apr 17 13:17:11 2025 from 10.190.238.102
root@n10:~# confd_client db_list  
- isc_dev_cloud_dwh
root@n10:~# confd_client db_stop db_name: isc_dev_cloud_dwh
OK
root@n10:~# confd_client db_state db_name: isc_dev_cloud_dwh
setup
root@n10:~# confd_client db_backup_list db_name: isc_dev_cloud_dwh show_foreign: True
- bid: 1
  comment: ''
  dependencies: '-'
  expire: 2025-04-24 14:56
  expire_alterable: 10002 isc_dev_cloud_dwh/id_1/level_0
  expired: false
  id: 10002 isc_dev_cloud_dwh/id_1/level_0/node_0/backup_202504171456 isc_dev_cloud_dwh
  last_item: false
  level: 0
  path: isc_dev_cloud_dwh/id_1/level_0/node_0/backup_202504171456
  system: isc_dev_cloud_dwh
  timestamp: 2025-04-17 14:56
  ts: '202504171456'
  usable: true
  usage: 0.001 GiB
  volume: default_backup_volume
- bid: 478
  comment: ''
  dependencies: '-'
  expire: 2025-04-20 10:00
  expire_alterable: 10003 isc_dev_cloud_dwh/id_478/level_0
  expired: false
  id: 10003 isc_dev_cloud_dwh/id_478/level_0/node_0/backup_202504061000 isc_dev_cloud_dwh
  last_item: false
  level: 0
  path: isc_dev_cloud_dwh/id_478/level_0/node_0/backup_202504061000
  system: isc_dev_cloud_dwh
  timestamp: 2025-04-06 10:00
  ts: '202504061000'
  usable: true
  usage: 165.784 GiB
  volume: r0000
- bid: 481
  comment: ''
  dependencies: '478'
  expire: 2025-04-16 22:30
  expire_alterable: 10003 isc_dev_cloud_dwh/id_481/level_1
  expired: true
  id: 10003 isc_dev_cloud_dwh/id_481/level_1/node_0/backup_202504082230 isc_dev_cloud_dwh
  last_item: false
  level: 1
  path: isc_dev_cloud_dwh/id_481/level_1/node_0/backup_202504082230
  system: isc_dev_cloud_dwh
  timestamp: 2025-04-08 22:30
  ts: '202504082230'
  usable: true
  usage: 44.720 GiB
  volume: r0000
- bid: 482
  comment: ''
  dependencies: '478'
  expire: 2025-04-17 22:30
  expire_alterable: 10003 isc_dev_cloud_dwh/id_482/level_1
  expired: false
  id: 10003 isc_dev_cloud_dwh/id_482/level_1/node_0/backup_202504092230 isc_dev_cloud_dwh
  last_item: false
  level: 1
  path: isc_dev_cloud_dwh/id_482/level_1/node_0/backup_202504092230
  system: isc_dev_cloud_dwh
  timestamp: 2025-04-09 22:30
  ts: '202504092230'
  usable: true
  usage: 45.376 GiB
  volume: r0000
- bid: 483
  comment: ''
  dependencies: '478'
  expire: 2025-04-18 22:30
  expire_alterable: 10003 isc_dev_cloud_dwh/id_483/level_1
  expired: false
  id: 10003 isc_dev_cloud_dwh/id_483/level_1/node_0/backup_202504102230 isc_dev_cloud_dwh
  last_item: false
  level: 1
  path: isc_dev_cloud_dwh/id_483/level_1/node_0/backup_202504102230
  system: isc_dev_cloud_dwh
  timestamp: 2025-04-10 22:30
  ts: '202504102230'
  usable: true
  usage: 47.413 GiB
  volume: r0000
- bid: 484
  comment: ''
  dependencies: '478'
  expire: 2025-04-18 22:30
  expire_alterable: 10003 isc_dev_cloud_dwh/id_484/level_1
  expired: false
  id: 10003 isc_dev_cloud_dwh/id_484/level_1/node_0/backup_202504112230 isc_dev_cloud_dwh
  last_item: false
  level: 1
  path: isc_dev_cloud_dwh/id_484/level_1/node_0/backup_202504112230
  system: isc_dev_cloud_dwh
  timestamp: 2025-04-11 22:30
  ts: '202504112230'
  usable: true
  usage: 47.988 GiB
  volume: r0000
- bid: 485
  comment: ''
  dependencies: '478'
  expire: 2025-04-18 22:30
  expire_alterable: 10003 isc_dev_cloud_dwh/id_485/level_1
  expired: false
  id: 10003 isc_dev_cloud_dwh/id_485/level_1/node_0/backup_202504122230 isc_dev_cloud_dwh
  last_item: false
  level: 1
  path: isc_dev_cloud_dwh/id_485/level_1/node_0/backup_202504122230
  system: isc_dev_cloud_dwh
  timestamp: 2025-04-12 22:30
  ts: '202504122230'
  usable: true
  usage: 48.525 GiB
  volume: r0000
- bid: 486
  comment: ''
  dependencies: '-'
  expire: 2025-04-27 10:00
  expire_alterable: 10003 isc_dev_cloud_dwh/id_486/level_0
  expired: false
  id: 10003 isc_dev_cloud_dwh/id_486/level_0/node_0/backup_202504131000 isc_dev_cloud_dwh
  last_item: false
  level: 0
  path: isc_dev_cloud_dwh/id_486/level_0/node_0/backup_202504131000
  system: isc_dev_cloud_dwh
  timestamp: 2025-04-13 10:00
  ts: '202504131000'
  usable: true
  usage: 166.442 GiB
  volume: r0000
- bid: 487
  comment: ''
  dependencies: '486'
  expire: 2025-04-21 22:30
  expire_alterable: 10003 isc_dev_cloud_dwh/id_487/level_1
  expired: false
  id: 10003 isc_dev_cloud_dwh/id_487/level_1/node_0/backup_202504132230 isc_dev_cloud_dwh
  last_item: false
  level: 1
  path: isc_dev_cloud_dwh/id_487/level_1/node_0/backup_202504132230
  system: isc_dev_cloud_dwh
  timestamp: 2025-04-13 22:30
  ts: '202504132230'
  usable: true
  usage: 34.476 GiB
  volume: r0000
- bid: 488
  comment: ''
  dependencies: '486'
  expire: 2025-04-22 22:30
  expire_alterable: 10003 isc_dev_cloud_dwh/id_488/level_1
  expired: false
  id: 10003 isc_dev_cloud_dwh/id_488/level_1/node_0/backup_202504142230 isc_dev_cloud_dwh
  last_item: false
  level: 1
  path: isc_dev_cloud_dwh/id_488/level_1/node_0/backup_202504142230
  system: isc_dev_cloud_dwh
  timestamp: 2025-04-14 22:30
  ts: '202504142230'
  usable: true
  usage: 36.045 GiB
  volume: r0000
- bid: 489
  comment: ''
  dependencies: '486'
  expire: 2025-04-23 22:30
  expire_alterable: 10003 isc_dev_cloud_dwh/id_489/level_1
  expired: false
  id: 10003 isc_dev_cloud_dwh/id_489/level_1/node_0/backup_202504152230 isc_dev_cloud_dwh
  last_item: false
  level: 1
  path: isc_dev_cloud_dwh/id_489/level_1/node_0/backup_202504152230
  system: isc_dev_cloud_dwh
  timestamp: 2025-04-15 22:30
  ts: '202504152230'
  usable: true
  usage: 37.368 GiB
  volume: r0000
- bid: 490
  comment: ''
  dependencies: '486'
  expire: 2025-04-24 22:30
  expire_alterable: 10003 isc_dev_cloud_dwh/id_490/level_1
  expired: false
  id: 10003 isc_dev_cloud_dwh/id_490/level_1/node_0/backup_202504162230 isc_dev_cloud_dwh
  last_item: true
  level: 1
  path: isc_dev_cloud_dwh/id_490/level_1/node_0/backup_202504162230
  system: isc_dev_cloud_dwh
  timestamp: 2025-04-16 22:30
  ts: '202504162230'
  usable: true
  usage: 38.089 GiB
  volume: r0000
...
```
-- Then stop the db and start the restore with the latest level 1 backup (should take about 27 min for 166 GB ):
```bash
root@n10:~# confd_client db_stop db_name: isc_dev_cloud_dwh
OK
root@n10:~# confd_client db_state db_name: isc_dev_cloud_dwh
setup
...
root@n10:~# confd_client db_restore db_name: isc_dev_cloud_dwh backup_id: '10003 isc_dev_cloud_dwh/id_490/level_1/node_0/backup_202504162230 isc_dev_cloud_dwh' restore_type: blocking
OK
root@n10:~# confd_client db_backup_progress db_name: isc_dev_cloud_dwh
Comment: Restore has been started
Files: []
Level: 1
Name: isc_dev_cloud_dwh/id_490/level_1/node_0/backup_202504162230
Progress: 0
Type: Restore
Volume ID: 10003
```
-- 27 min later:

```bash
root@n10:~# confd_client db_backup_progress db_name: isc_dev_cloud_dwh
Comment: Restore has been successfully finished
Files: []
Level: 1
Name: isc_dev_cloud_dwh/id_490/level_1/node_0/backup_202504162230
Progress: 100
Type: Restore
Volume ID: 10003
root@n10:~# confd_client db_state db_name: isc_dev_cloud_dwh
running
...
```
-- Later the database will be running but still needs to wait (5-10 min) until the data nodes get to stage d
```bash
dw_etl@isc-pdap-cloudetl11:~/GIT/py_database_tools> c4 ps
      N  PLAY_ID   NODE  MEDIUM  INSTANCE      DB_VERSION  EXTERNAL_IP  INTERNAL_IP    STAGE  STATE    UPTIME    TTL  
  ┌─  1  1e74b46c  10    awscf   c5ad.large    8.33.0      -            10.190.238.58  c      running  01:10:11  +∞   
  │   1  1e74b46c  11    awscf   c5ad.4xlarge  8.33.0      -            10.190.238.40  d      running  01:10:11  +∞   
  └─  1  1e74b46c  12    awscf   c5ad.4xlarge  8.33.0      -            10.190.238.25  d      running  01:10:11  +∞   
dw_etl@isc-pdap-cloudetl11:~/GIT/py_database_tools> 
```
-- Then to log in you will need the user: sys pasword: Exasol@AWS Database Password in BI_common.idkb 
(with the connection_string = 10.190.238.25,10.190.238.40) and later configure de LDAP loginto use you user and pw.



