# Proxmox VM creation with CloudImage and CloudInit

I needed to build a few VMs for my k8s cluster - a master node which would hold the k8s control plane and a few worker nodes.

I decided that I didn't fancy building 4 VMs for my k8s cluster by installing the O/S (Ubuntu 24.04) from the ISO and going through the installation GUI when faster methods were available.

Deploying templates to VMs using CloudInit starts with the creation of the VM template.  While this can be done via the Proxmox UI, I chose to use the command-line as much as possible.

To create the template, there are a number of internet resources you can follow.  I based myself on Techno Tim's [Perfoct Proxmox Template with Cloud Image and Cloud Init video](https://www.youtube.com/watch?v=shiIi38cJe4).  While the video is around 3 years old, I found it a good base.  Note that Tim also publishes a [blog](https://technotim.live/posts/cloud-init-cloud-image/) with the details of the commands etc.

All the commands below were run as the root user on Proxmox.

## The key steps are:
1. Download the cloud image
2. Create the new Virtual machine
3. Import then attach the downloaded CloudInit image to the VM
4. Add a Cloudinit drive to the VM and make it bootable
5. Add a serial console to the VM
6. Convert the VM to a template
7. Configure CloudInit (the parameters, that is) 
8. Clone the template into a new VM
9. Update the CloudInit configuration in the VM to meet your needs
10. (Optional) Grow the disk


### Download the CloudInit image
The Cloud Init image is similar to an ISO file and is the boot/installation disk for our VM. 
Canonical's repository of Cloud Init images is [here](https://cloud-images.ubuntu.com/)
You will need to download the image that suits you needs.  For me that was:
https://cloud-images.ubuntu.com/noble/current/noble-server-cloudimg-amd64.img

To perform the download, use this command (replacing the URL with your choice of CloudInit image) while logged into your proxmox server as root.  The file will be downloaded in the current directory, though you may wish to download it to `/var/lib/vz/template/iso`.  In a later step it will be imported into the storage of your choice.

```bash
wget https://cloud-images.ubuntu.com/noble/current/noble-server-cloudimg-amd64.img
```

### Create the new VM (to later be converted to a template)
In this process, we first create a VM and later, when we want to use it, we clone it.  As you build this, it's best practice to convert the VM into a "template" which protects you from accidentally starting it, etc.  It is the template that we will (later) clone into the actual VM we want.

Pick the values you need for memory, CPU, etc., making sure that the VM ID is unique.  
<pre>
qm create 8200 --memory 4096 --core 4 --name ubuntu-cloud --net0 virtio,bridge=vmbr0
</pre>

This creates a VM with ID 8200, sets the core count to 4, gives it the name "ubuntu-cloud" and sets the first network interface to use the virtio driver and the `vmbr0` 'bridge'.

### Import then attach the downloaded CloudInit image to the VM
For it to be used by the VM, the disk must be imported to the proxmox storage.  On my server, that storage is called "data4tb".  This imported disk must then be associated with the VM.

The two commands are below - The bolded text needs to be adapted for your environment.

"noble-server-cloudimg-amd64.img" is the file name from the wget command which will be imported into the Proxmox VM as vm-8200-disk-0 because it's disk 0 of VM id 8200.
<pre>
qm disk import 8200 noble-server-cloudimg-amd64.img <b>data4tb</b>
qm set 8200 --scsihw virtio-scsi-pci --scsi0 <b>data4tb</b>:vm-8200-disk-0
</pre>

### Add a Cloudinit 'drive' to the VM and make it bootable
The cloudinit drive (usually an ISO file or simiar) contains the parameters for the cloudinit operation that will take place when the VM boots.
The parameters/values can be set in the Proxmox GUI:
![Alt](/articles/assets/VM_Cloudinit_parameters.png)

Note that the first command will create the cloud-init disk - though I think it's actually an ISO file rather than a generic disk.
While the 2nd command will set the boot disk to the volume attached to the scsi0 interface which is where the downloaded cloudinit disk was mounted.

<pre>
qm set 8200 --ide2 <b>data4tb</b>:cloudinit
qm set 8200 --boot c --bootdisk scsi0
</pre>


### Add a serial console to the VM
This step is need to enable you to view boot output, etc. via Proxmox's VNC (a virtual remote/screen).

If you haven't added SSH keys, this step is vital.

<pre>
qm set 8200 --serial0 socket --vga serial0
</pre>

### Convert the VM to a template
### Configure CloudInit (the parameters, that is) 
### Clone the template into a new VM
### Update the CloudInit configuration in the VM to meet your needs

### (Optional) Grow the disk.
When the disk is imported in the step "Import then attach the downloaded CloudInit image to the VM", the size is just enough to hold its content.  Consuming even a relatively small amount of storage can fill the disk which is never a good thing in *nix.

Below is a command that should resize the disk by 50Gb.  I have not tested this and did it throught the GUI.
<pre>
qm resize 210 scsi0 +50G
</pre>

