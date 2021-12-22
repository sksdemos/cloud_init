# Ansible role `cloud_init` v0.1

Ansible role that acts as a provisioner  for creating multiple virtual machines at a blazing fast pace. This role leverages on the efficiencies of libvirt and qemu packages along with cloud-init to create VMs. Cloud-init is the **industry standard multi-distribution method for cross-platform cloud instance initialization**. It is supported across all major public cloud providers, provisioning systems for private cloud infrastructure, and bare-metal installations. The role is optimized in many ways:

 - Prevents multiple downloads of the same cloud image
 - Prevents re-creation of VMs by stalling the role if they already exist. This behavior can be changed
 - For slow connections the cloud image can be downloaded and put in a local folder to prevent downloading from the web

## Requirements
This role has the following requirements:
 - qemu-kvm, qemu-img, genisoimage and libguestfs-tools package
 - active libvirt storage pool and network previously provisioned if they are required by the VMs

While not a role dependency the above requirements can be easily met if the libvirt_setup role is used.

## Role Variables
This role requires a set of variables along with the dictionary  **cloud_vms_info**  in role/defaults that require to be overridden to create a set of VMs. The dictionary uses a key that acts as the VMs name and a set of VM related parameters to be used on a per VM basis. 

### cloud_vms_info.<VM_NAME> dictionary variables
This dictionary is used initialize when a new network is required for VMs for a specific deployment that need to use a specific IP address range that does not interfere with existing applications.
| Variable | Comments |
| :---  | :--- |
| vm_image_link | URL for a specific cloud image
| vm_local_hostname | Hostname assigned to the VM during the cloud-init process.
| vm_fqdn | Fully qualified domain name of the Virtual machine. Used by cloud-init
| vm_root_passwd | Password to set for the root user during the cloud-init process
| vm_os_type | Type of the OS being installed. i.e. **windows** or **linux**
| vm_os_variant | Variant of the OS being installed. Run the **virt-install --os-variant list** command to see a set of supported OS variants.
| vm_memory | Memory required by the VM.
| vm_cpu | Number of vCPUs required by the VM
| vm_root_disk_size | Total size of the block device
| vm_static_ip | Static IP to be assigned to the VM. (optional)
| vm_netmask | Subnet mask to be used with the static IP. Must be set if static IP is being used
| vm_gateway | The default gateway that the VM will use to access external networks. Must be set if static IP is used
| vm_dns1 | First DNS server IP address. Can be used when setting a static IP to point to a local DNS

### Other role variables
|Variable|Comments  |
|--|--|
| vm_recreate | Defaults to a value of **False** for safety reasons so that VMs are not recreated if they are already existing. This causes your role to stall with a message. Can be set to **True** to delete the VM and recreate it again|
| cloud_images_dir | Defaults to the name **cloud_images**. Will be created in the same place as the playbook that calls the role, if it doesn't exist. If this folder is already existing then the role tries to rsync the folder content to the path stored in the cloud_storage_path |
| cloud_storage_path | Path where the cloud image is stored and VM boot disk is created. |
| cloud_network | Libvirt network to be used by cloud images. This defaults to the default libvirt network called **default**. |
| cloud_image_download_timeout| Max time to download cloud image. Defaults to 300 seconds. Increase this value for slower internet connections. |
| cloud_image_download_poll | Polls for task completion every 10 seconds by default. |

#### Example. 
Creates a VM called test1 that uses a static IP address.
```Yaml
cloud_vms_info:
  test1:
    vm_image_link: http://cloud.centos.org/centos/7/images/CentOS-7-x86_64-GenericCloud-2009.qcow2
    vm_local_hostname: test1
    vm_fqdn: test1.example.com
    vm_root_passwd: letmein
    vm_os_type: linux
    vm_os_variant: rhel7.0
    vm_memory: 1024
    vm_cpu: 1
    vm_root_disk_size: 10G
    vm_static_ip: 192.168.122.250
    vm_netmask: 255.255.255.0
    vm_gateway: 192.168.122.1
#   vm_dns1:
```
> **NOTE**: If the above list of dictionaries are not specified while executing the role then the role will create two test VMs called test1 and test2 based on the dictionary in role defaults. One will function using a static IP address the other will take a DHCP lease from the default libvirt network. Both VM install disks will be located in the default storage pool.

#### Example. 
 Creates a VM called test2 that uses a DHCP lease from the libvirt network.

```Yaml
cloud_vms_info:
  test2:
    vm_image_link: http://cloud.centos.org/centos/7/images/CentOS-7-x86_64-GenericCloud-2009.qcow2
    vm_local_hostname: test2
    vm_fqdn: test2.example.com
    vm_root_passwd: letmein
    vm_os_type: linux
    vm_os_variant: rhel7.0
    vm_memory: 1024
    vm_cpu: 1
    vm_root_disk_size: 10G
```
## Dependencies
No dependencies.

## License
BSD

## Author
**Sanjay Singh**
sanjay.kr.singh@gmail.com
> Written with [StackEdit](https://stackedit.io/).
