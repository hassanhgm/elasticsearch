
# Planning for Elasticsearch
##	Understand the Indexing requirements.
	*	Size and attributes of documents for Index mappings
	*	Expectation on # of Indexing documents/sec (TPS).
	*	Setup Index and aliases.
	*	Setup Index Lifecycle policy.
		*	Hot Index-->​ the index is actively being updated and queried.
		*	Warm Index--> the index is no longer being updated, but is still being queried.
		*	Cold Index--> the index is no longer being updated and is seldom queried. The information still needs to be searchable, but it’s okay if those queries are slower.
		*	Delete Index--> ​the index is no longer needed and can safely be deleted.
		
##	Understand the Search requirements.
	*   Set expectation and benchmarking Search criteria for 90, 95 and 99% in ms.
	*   Understanding the types and pattern of search queries.
	*   Search vs Indexing to be understood.
	
##	Have any stats to benchmark the new Elasticsearch ?
	* Check and document if you have any metrics to benchmark Elasticsearch 
	
##	Plan for Cluster design.
    * Capacity planning for atleast 3yrs of Data Load.
        * Avg day data growth * 30 * 12 for Primary + # Replicas
        
    * Discuss on Type of VM Series(Preferred E and L Series) for Data Nodes and size. 
    * Discuss on Type of VM Series(Preferred D Series) for Master Nodes and size. 
    * Setup atleast three master nodes in Master Availaibity Set.
    * Setup atleast five Data nodes in Data Availaibity Set.
    * Setup atleast two client nodes in Client Availaibity Set.
    * Setup Azure Internal Load Balancer with static IP.
    * Allow 9200;9300 and 5601 Ports in NSG.
	
## Plan for Shards and replicas.

    * Lucene hard limit is set 2^32 documents or <=50 GB per shard. Based on your data growth projection of atleast a year
      you should do the math to have number of shards created per index.
    * # of replicas improve you search performance and availabity. But it is increase the storage cost.
    * Set index life cycle policy to better manage ever growing indexes.

## Security
    * Always use CIS certified tesco shared Images for OS.
    * Deny all other rules from NSG other than 9200 and 5601.
    * Encrption of Disk/VM OS Disk should be done  before installation of Elasticsearch and Kibana.
    * If using open source version of Elasticsearch along with Kibana, Please enable SSL and RBAC using 
      XPack.
    
# Installation of Elasticsearch in Azure

    * Start setting up Availabilty Sets 
       * Master  Availability Set
       * Data    Availability Set
       * Client  Availability Set
       * Kibana  Availability Set
    * Install CentOs VMs with shared immutable image with data and log disk 
      in AvSets for Data Nodes.
    * Install CentOs VMs with shared immutable image with log disk 
      in AvSets for Master Nodes.
    * Install CentOs VMs with shared immutable image with log disk 
      in AvSets for Client Nodes.
    * Install CentOs VMs with shared immutable image with log disk 
      in AvSets for Kibana Nodes.        
    * Encrypt data drives and log drives of all nodes.
    * Setup standard Internal Load Balancer with Static IP.

# Harden OS CentOS for Elasticsearch and Kibana in Azure.

* Set swappiness to 0 or 1
* Set file Descriptors to 64000
* Set max map memory count
* Set ulimit to 65535 for elasticsearch user account
* Install java (preffered latest jkd)

## Run below script to set all the hardening values 

    * Create os_config.sh file copy below code with execute permisson and run as sudo su
    
          echo " Report swappiness set to 0"
          sudo sh -c 'echo 0 > /proc/sys/vm/swappiness'
          
          
          echo " Seting swappiness to 0"
          # Set the value in /etc/sysctl.conf so it stays after reboot.
          sudo sh -c 'echo "" >> /etc/sysctl.conf'
          sudo sh -c 'echo "#Added By S.Hassan to set swappiness to 0 to avoid swapping" >> /etc/sysctl.conf'
          sudo sh -c 'echo "vm.swappiness = 0" >> /etc/sysctl.conf'
          
          echo " Set file Descriptors to 64000 for elasticsearch"
          
          # Set the value in /etc/sysctl.conf so it stays after reboot.
          sudo sh -c 'echo "" >> /etc/sysctl.conf'
          sudo sh -c 'echo "#Added By <Author Name> to set File Descriptors settings count to 64000 to avoid too many files open errors msg" >> /etc/sysctl.conf'
          sudo sh -c 'echo "fs.file-max=64000" >> /etc/sysctl.conf'
          
          echo " Set max map memory" 
          
          # Set the value in /etc/sysctl.conf so it stays after reboot.
          sudo sh -c 'echo "" >> /etc/sysctl.conf'
          sudo sh -c 'echo "#Added By <Author Name> to set mmap count to 262144 to avoid memory mapping issues" >> /etc/sysctl.conf'
          sudo sh -c 'echo "vm.max_map_count = 262144" >> /etc/sysctl.conf'
          
          echo " Set ulimit t0 65535"
          # Set the value in /etc/sysctl.conf so it stays after reboot.
          sudo sh -c 'echo "" >> /etc/security/limits.conf'
          sudo sh -c 'echo "#Added By <Author Name> to set sufficient threads for open file handlers for elasticsearch" >> /etc/security/limits.conf'
          sudo sh -c 'echo "elasticsearch  -  nofile  65535" >> /etc/security/limits.conf'
          
          echo" Install java 11"
          yum install java-11-openjdk-devel.x86_64 -y
          echo " Reboot System to verify the system configuration"      
      
      * Once rebooted verfiy the above settings.
      
            sudo cat /proc/sys/vm/swappiness 
            cat /proc/sys/vm/max_map_count 
            sysctl fs.file-max 
            java --version
            

## Format and mount Data and Log drive onto VMs
ssh to VMs

sudo su
    
    dmesg | grep SCSI
         # Look for /dev/sdc or /dev/sdd soon
         
         [    0.371055] SCSI subsystem initialized
         [    0.973225] Block layer SCSI generic (bsg) driver version 0.4 loaded (major 248)
         [    3.659635] sd 3:0:1:0: [sdb] Attached SCSI disk
         [    3.668895] sd 2:0:0:0: [sda] Attached SCSI disk
         [   22.155070] sd 5:0:0:0: [sdc] Attached SCSI disk
    
fdisk -l   
        
           To list out all the drives

Output: Look for /dev/sdc and /dev/sdd with the size. These are the disk which we would
        like to format and mount.
            $fdisk -l 
            
                $ sudo fdisk -l 
                
                 Output
                 
                    Disk /dev/sdb: 107.4 GB, 107374182400 bytes, 209715200 sectors
                    Units = sectors of 1 * 512 = 512 bytes
                    Sector size (logical/physical): 512 bytes / 4096 bytes
                    I/O size (minimum/optimal): 4096 bytes / 4096 bytes
                    Disk label type: dos
                    Disk identifier: 0x45e9e261
                    
                     
                    
                       Device Boot      Start         End      Blocks   Id  System
                    /dev/sdb1            2048   209713151   104855552   83  Linux
                    
                     
                    
                    Disk /dev/sda: 34.4 GB, 34359738368 bytes, 67108864 sectors
                    Units = sectors of 1 * 512 = 512 bytes
                    Sector size (logical/physical): 512 bytes / 512 bytes
                    I/O size (minimum/optimal): 512 bytes / 512 bytes
                    Disk label type: dos
                    Disk identifier: 0x000769d6
                    
                     
                    
                       Device Boot      Start         End      Blocks   Id  System
                    /dev/sda1   *        2048     2099199     1048576   83  Linux
                    /dev/sda2         2099200    33556479    15728640   83  Linux
                    /dev/sda3        33556480    46139391     6291456   83  Linux
                    /dev/sda4        46139392    67108863    10484736    5  Extended
                    /dev/sda5        46143488    54532095     4194304   83  Linux
                    /dev/sda6        54534144    58728447     2097152   83  Linux
                    /dev/sda7        58730496    60827647     1048576   83  Linux
                    /dev/sda8        60829696    62926847     1048576   83  Linux
                    /dev/sda9        62928896    65026047     1048576   83  Linux
                    
                   Disk /dev/sdc: 34.4 GB, 34359738368 bytes, 67108864 sectors
                   Units = sectors of 1 * 512 = 512 bytes
                   Sector size (logical/physical): 512 bytes / 4096 bytes
                   I/O size (minimum/optimal): 4096 bytes / 4096 bytes
         
            # Format the data disk with start block to 4096 and log disk to default which is 2048
          
                $ sudo fdisk /dev/sdc   
                
                
                    Command (m for help): n
                    Partition type:
                       p   primary (0 primary, 0 extended, 4 free)
                       e   extended
                    Select (default p): p
                    Partition number (1-4, default 1): 1
                    First sector (2048-67108863, default 2048): 4096
                    Last sector, +sectors or +size{K,M,G} (4096-67108863, default 67108863): 
                    Using default value 67108863
                    Partition 1 of type Linux and of size 32 GiB is set
                    
                    
                    Command (m for help): p
                    
                    Disk /dev/sdc: 34.4 GB, 34359738368 bytes, 67108864 sectors
                    Units = sectors of 1 * 512 = 512 bytes
                    Sector size (logical/physical): 512 bytes / 4096 bytes
                    I/O size (minimum/optimal): 4096 bytes / 4096 bytes
                    Disk label type: dos
                    Disk identifier: 0x810eb55b
                    
                       Device Boot      Start         End      Blocks   Id  System
                    /dev/sdc1            4096    67108863    33552384   83  Linux
                    
                    
                    
                    Command (m for help): w
                    The partition table has been altered!
                    
                    Calling ioctl() to re-read partition table.
                    Syncing disks.
                   
                    
### Write a file system to the partition with the mkfs command.
            
* sudo mkfs -t ext4 /dev/sdc1  
           
           $ sudo mkfs -t ext4 /dev/sdc1
           
           Output
           
               mke2fs 1.42.9 (28-Dec-2013)
               Discarding device blocks: done                            
               Filesystem label=
               OS type: Linux
               Block size=4096 (log=2)
               Fragment size=4096 (log=2)
               Stride=0 blocks, Stripe width=0 blocks
               2097152 inodes, 8388096 blocks
               419404 blocks (5.00%) reserved for the super user
               First data block=0
               Maximum filesystem blocks=2155872256
               256 block groups
               32768 blocks per group, 32768 fragments per group
               8192 inodes per group
               Superblock backups stored on blocks: 
               32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208, 
               4096000, 7962624
               
               Allocating group tables: done                            
               Writing inode tables: done                            
               Creating journal (32768 blocks): done
               Writing superblocks and filesystem accounting information: done
            
                    
### Create a directory to mount the file system using mkdir and mount the drive.

                $ sudo mkdir /datadrive  

                $ sudo mount /dev/sdc1 /datadrive
            
            # Verify the drive is correctly mounted
            
                $ df -hT
                
                    Filesystem     Type      Size  Used Avail Use% Mounted on
                    devtmpfs       devtmpfs  3.9G     0  3.9G   0% /dev
                    tmpfs          tmpfs     3.9G     0  3.9G   0% /dev/shm
                    tmpfs          tmpfs     3.9G  9.0M  3.9G   1% /run
                    tmpfs          tmpfs     3.9G     0  3.9G   0% /sys/fs/cgroup
                    /dev/sda2      xfs        30G  2.1G   28G   8% /
                    /dev/sda1      xfs       497M   64M  433M  13% /boot
                    /dev/sdb1      ext4       16G   45M   15G   1% /mnt/resource
                    tmpfs          tmpfs     797M     0  797M   0% /run/user/1000
                    /dev/sdc1      ext4       32G   49M   30G   1% /datadrive
             
             # Change the ownership of mount drive
                
                    chown -R elasticsearch /datadrive
                    chown -R elasticsearch /logdrive
                    
                    chgrp elasticsearch /datadrive
                    chgrp elasticsearch /logdrive
               
### To ensure that the drive is remounted automatically after a reboot, it must be added to the /etc/fstab file.

 Note: It is also highly recommended that the UUID (Universally Unique IDentifier) is used in /etc/fstab to refer to the drive rather than 
       just the device name (such as, /dev/sdc1).
       
                 $ sudo -i blkid
                       
                   /dev/sda1: UUID="8aae9743-cbcf-4efc-a226-04414bc81b94" TYPE="xfs" 
                   /dev/sda2: UUID="17199881-6ce3-4652-9a93-94ac34384ae6" TYPE="xfs" 
                   /dev/sdb1: UUID="236a2e83-4813-4bb1-bc10-31a404dfafb8" TYPE="ext4" 
                   /dev/sdc1: UUID="560e8295-f182-403a-84e9-e7d0fd09d90e" TYPE="ext4"  
                   
             # We did formatting of /dev/sdc volume and it created a primary partition of type /dev/sdc1.
             # choose UUID of /dev/sdc1 and embed in below command and run.
             
                 $ sudo sh -c 'echo "UUID=560e8295-f182-403a-84e9-e7d0fd09d90e /datadrive              ext4   defaults,nofail   1   2" >> /etc/fstab'               
   

## Encrpyt Data and Log drives on all VMs.
    Note: Should have keyvault and enabled for Azure Disk encryption.
    
    
    * Create  KEK (Key Encryption Key for VM)
    
        az keyvault key create --name "es-disk-encr-key-delete-me" --vault-name "euw-dev-127-cos-inf-kv" --kty RSA --subscription ff0d466c-fe1d-495b-810a-73fb6fa79e04 --size 4096 --ops decrypt encrypt sign unwrapKey verify wrapKey 
    
    
    * Azure cli command to encrypt.
    
        az vm encryption enable --resource-group "MyVirtualMachineResourceGroup" 
        --name "MySecureVM" 
        --disk-encryption-keyvault "<ResourceID of keyvault>"
        --key-encryption-key "MyKEK_URI" 
        --key-encryption-keyvault "<ResourceID of keyvault>" 
        --volume-type [All|OS|Data]
    
    Example:
    
        az vm encryption enable --resource-group "euw-dev-127-cos-es-rg" --name "euw-dev-127-cos-es-master0-vm" --disk-encryption-keyvault "/subscriptions/ff0d466c-fe1d-495b-810a-73fb6fa79e04/resourceGroups/euw-dev-127-cos-inf-rg/providers/Microsoft.KeyVault/vaults/euw-dev-127-cos-inf-kv" --key-encryption-key "https://euw-dev-127-cos-inf-kv.vault.azure.net/keys/127-centos-vm-disk-encrypt/fa70a8334e534e479cfb307aa5c72985" --key-encryption-keyvault "/subscriptions/ff0d466c-fe1d-495b-810a-73fb6fa79e04/resourceGroups/euw-dev-127-cos-inf-rg/providers/Microsoft.KeyVault/vaults/euw-dev-127-cos-inf-kv" --volume-type Data     

az vm encryption show --resource-group "euw-dev-127-cos-es-rg" --name "euw-dev-127-cos-es-data0-vm"
## Install Elasticsearch on all nodes (master,data and client)
* ssh VMs 
    
Download stable elasticsearch RPM
  
    Note : We are doing for elasticsearch 7.0.0. Remains same for any version.
 
* Copy below code and run in all VMs. This will install and harden the ES. But will not configure cluster.
        
            Create a file InstallConfigES.sh to install and harden Elasticsearch 
            
            #============================================Installation Scrpits for Elastic Search======================================================
            
            echo " Downloading Elasticsearch RPM package"
            wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.0.0-x86_64.rpm
            sleep 5s
            
            echo " Siging in Elasticsearch GPG Key"
            rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch
            echo ""
            
            echo " Installating Elasticsearch"
            sudo rpm -ivh elasticsearch-7.0.0-x86_64.rpm
            echo " Installation completed"
            
            sleep 5s
            echo " Daemon reload "
            systemctl daemon-reload
            echo " Daemon reload completed"
            
            echo " Register and enable elasticsearch service"
            systemctl enable elasticsearch.service
            
            sleep 5s
            echo " Starting Elasticsearch Service"
            #systemctl start elasticsearch.service
            
            sleep 15s
            echo " Verifying Status of Elasticsearch Service" 
            echo ""
            systemctl status elasticsearch.service
            
            sleep 5s
            echo " Stopping Elasticsearch Service"
            echo ""
            systemctl stop elasticsearch.service
            
            #==============================================Configuration of Elasticsearch==============================================================
            
            echo " Set JVM.Option min and max memory to 50% of total RAM"
            #!/bin/bash
            memoryInKb="$(awk '/MemTotal/ {print $2}' /proc/meminfo)"
            heapSize="$(expr $memoryInKb / 1024 / 1000 / 2)"
            sed -i "s/#*-Xmx[0-9]\+g/-Xmx${heapSize}g/g" /etc/elasticsearch/jvm.options
            sed -i "s/#*-Xms[0-9]\+g/-Xms${heapSize}g/g" /etc/elasticsearch/jvm.options
            echo " "
            
            sleep 5s
            
            echo " Configuring elasticsearch.service in /etc/systemd settings on LimitMEMLOCK,LimitNOFILE & Elasticsearch Thread pool (LimitNPROC)"
            echo " "
            sudo sh -c 'echo "" >> /usr/lib/systemd/system/elasticsearch.service'
            sudo sh -c 'echo "#Added By S.Hassan to set mlock,flie descriptor on elasticsearch" >> /usr/lib/systemd/system/elasticsearch.service'
            sudo sh -c 'echo "[Service]" >> /usr/lib/systemd/system/elasticsearch.service'
            sudo sh -c 'echo "LimitMEMLOCK=infinity" >> /usr/lib/systemd/system/elasticsearch.service'
            sudo sh -c 'echo "LimitNPROC=4096" >> /usr/lib/systemd/system/elasticsearch.service'
            sudo sh -c 'echo "LimitNOFILE=65536" >> /usr/lib/systemd/system/elasticsearch.service'
            echo " "
            
            sleep 5s
            echo " Daemon reload "
            systemctl daemon-reload
            echo " Daemon reload completed"
            
            echo " Configuring elasticsearch settings in /etc/sysconfig/ on MAX_LOCKED_MEMORY, MAX_MAP_COUNT & MAX_OPEN_FILES"
            echo " "
            sudo sh -c 'echo "" >> /etc/sysconfig/elasticsearch'
            sudo sh -c 'echo "#Uncommented By S.Hassan to set MAX_LOCKED_MEMORY,MAX_OPEN_FILES & MAX_MAP_COUNT " >> /etc/sysconfig/elasticsearch'
            sudo sh -c 'echo "MAX_LOCKED_MEMORY=unlimited" >> /etc/sysconfig/elasticsearch'
            sudo sh -c 'echo "MAX_OPEN_FILES=65536" >> /etc/sysconfig/elasticsearch'
            sudo sh -c 'echo "MAX_MAP_COUNT=262144" >> /etc/sysconfig/elasticsearch'
            sudo sh -c 'echo "" >> /etc/sysconfig/elasticsearch'
            
            sleep 5s
            echo " Daemon reload "
            systemctl daemon-reload
            echo " Daemon reload completed"
            
            echo "Stopping Elasticsearch Service"
            echo ""
            systemctl stop elasticsearch.service
            
            sleep 5s
            echo "Starting Elasticsearch Service"
            #systemctl start elasticsearch.service
 
## Configure Elasticsearch Cluster.


    
    