![Alt](/articles/assets/proxmox-server-solutions-gmbh-logo-vector.png) ![Alt](/articles/assets/cloud-init-logo-vector.png)

# Proxmox VM creation with CloudImage and CloudInit

I wanted to build a few VMs for my k8s cluster - a master node which would hold the k8s control plane and a few worker nodes.

I decided that I didn't fancy building 4 VMs for my k8s cluster by installing the O/S (Ubuntu 24.04) from the ISO and going through the installation GUI when faster methods were available.  Enter CloudInit.

Deploying templates to VMs using [CloudInit](https://cloud-init.io/) starts with the creation of the VM template.  While this can be done via the Proxmox UI, I chose to use the command-line as much as possible.

To create the template, there are a number of internet resources you can follow.  I based myself on Techno Tim's [Perfoct Proxmox Template with Cloud Image and Cloud Init video](https://www.youtube.com/watch?v=shiIi38cJe4) though there are a number of very similar videos and blogs.  While the video is around 3 years old, I found it a good base.  Note that Tim also publishes a companion [blog](https://technotim.live/posts/cloud-init-cloud-image/) with the details of the commands etc.

All the commands below were run as the root user on Proxmox (or "pve" for Proxmox Virtual Environment).

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

To perform the download, use this command (replacing the URL with your choice of CloudInit image) while logged into your proxmox server as root.  The file will be downloaded in the current directory, though you may wish to download it to `/var/lib/vz/template/iso` (the iso area of the 'local' storage in Proxmox).  In a later step the downloaded image will be imported into the storage of your choice alongside the VM definition.

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
The parameters/values can be set in the Proxmox GUI and this is done later in this process:
![Alt](/articles/assets/VM_Cloudinit_parameters.png)

Note that the first command will create the cloud-init disk - though it's actually an ISO file rather than a generic disk.  It will output the progress of the creation of the disk.
The 2nd command will set the boot disk to the volume attached to the scsi0 interface which is where the downloaded cloudinit disk was mounted.


TODO - Move the boot command to an earlier section

<pre>
qm set 8200 --ide2 <b>data4tb</b>:cloudinit
qm set 8200 --boot c --bootdisk scsi0
</pre>


### Add a serial console to the VM
This step is needed to enable you to view boot output, etc. via Proxmox's Console function (VNC - a virtual remote/screen).

If you you don't plan on adding some SSH keys, this step is vital.

<pre>
qm set 8200 --serial0 socket --vga serial0
</pre>

### Convert the VM to a template
While you can clone a VM, it's best practice to create templates (as above) and to clone the template.

<pre>
qm clone 8200
</pre>

### Configure CloudInit (the parameters, that is) 
The cloudinit process which is processed on every start of a cloudinit VM takes its configuration from the configuration data stored in the cloudinit disk we created earlier.

You can manually configure the cloudinit parameters using the GUI as mentioned earlier.  However the intent of this article is to document the steps you will need to create the VMs using just the command line so that, in time, we can automate the process via a script or an automation tool.

Certain aspects of cloudinit can be configured individually on the command-line.  e.g. you can set the ssh key to the contents of a file thus: `qm set 8200 --sshkey ~/.ssh.id_rsa.pub` which will set the SSH key to the contents of the id_rsa.pub (the public key) from the .ssh directory of the user's home directory.

In my lab, I need to be able to ssh into the VMs from multiple different places - my home computer, the Proxmox CLI and from my Ansible server and for this I need set the cloudinit all three keys.

The easiest way to do this is to update the 'user' section of cloudinit which includes configuration of the user, their password, and other parameters including the various ssh keys that will be needed.  Note that you only deploy the public keys!

To do this, we need to create a yaml configuration file.  For my example, the yaml file will look like this:
```yaml
#cloud-config
hostname: ubuntu-cloud
manage_etc_hosts: true
fqdn: ubuntu-cloud.lab.davidmjudge.me.uk
user: ansibleuser
ssh_authorized_keys:
  - ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIKL1EWv5ZwWTti7qoZbA+OZDGE5U+JhUU1Mxb+M0ZxkL ansibleuser@ansible4
  - ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIAP9eyxA4P8mE51qmbnigiuEmX72dRFRuN4SLmp0ISuA david@Ryzen2
chpasswd:
  expire: False
users:
  - default
package_upgrade: true
```

TODO - Rewrite this paragraph

#### Creating a snippets directory
This yaml file can be stored in the pve user's home directory.  But a better place to save such small configuration files is as a "snippet".  To be able to create a snippet, you will first need to add "snippet" to the storage in which you want to store the snippet.  To do this, in the Proxmox GUI, select your storage and add a directory to it:
![Alt](/articles/assets/Creating_Snippets_Directory.png)

I created my "snippets" directory as "/snippets" and that is the (root-level) folder where I will place snippets.  As I expect to use many (similar) snippets, I will create a directory for this aspect of my project and home lab which I will call `k3s-cloudinit` and thus will store my snippets in `/snippets/k3s-cloudinit`.  One of the benefits of creating this "snippets" directory as described here is tat it is very easy to find and to reference.  Also, we can use SCP to copy a file from my machine to pve, or even keep the snippets in a github that one can clone into the snippets

While there are 4 different areas in cloudinit (user, network and meta), I will only need to customise 2 of them: user and network.


You can then use the yaml files for user and network to configure those aspect of cloudinit.

Let's create the two files.
First the file for the user section:
```bash
cat >> /snippets/k3s-cloudinit/user-data.yaml <<EOF
#cloud-config
hostname: ubuntu-cloud
manage_etc_hosts: true
fqdn: ubuntu-cloud.lab.davidmjudge.me.uk
user: ansibleuser
ssh_authorized_keys:
  - ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIKL1EWv5ZwWTti7qoZbA+OZDGE5U+JhUU1Mxb+M0ZxkL ansibleuser@ansible4
  - ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIAP9eyxA4P8mE51qmbnigiuEmX72dRFRuN4SLmp0ISuA david@Ryzen2
chpasswd:
  expire: False
users:
  - default
package_upgrade: true
EOF
```

The the file for the network section:
```bash
cat >> /snippets/k3s-cloudinit/network-data.yaml <<EOF
version: 1
config:
    - type: physical
      name: eth0
      mac_address: 'bc:24:11:39:d8:2a'
      subnets:
      - type: static
        address: '192.168.178.210'
        netmask: '255.255.255.0'
        gateway: '192.168.178.1'
    - type: nameserver
      address:
      - '192.168.178.1'
      search:
      - 'lab.davidmjudge.me.uk
EOF
```

The next command will configure cloudinit for our template with the 2 sections.  In the command I will reference the snippets using the "snippets" storage reference `snippets:`.

```bash
qm  set 8200  --cicustom "user=snippets:k3s-cloudinit/user-data.yaml,network=snippets:k3s-cloudinit/network-data.yaml.yaml"
```

I expect that when I get around to using this, I will actually want to have configurations files that are variations on a theme... I can't, for example, configure 3 k3s worker nodes with the same name and IP addresses - I will create a script to generate a pair of files (user and network) for each of my worker nodes.
As soon as I start doing this, I will update this article with more detail on how this was done.

### Clone the template into a new VM
### Update the CloudInit configuration in the VM to meet your needs

### (Optional) Grow the disk.
When the disk is imported in the step "Import then attach the downloaded CloudInit image to the VM", the size is just enough to hold its content.  Consuming even a relatively small amount of storage can fill the disk which is never a good thing in *nix.

Below is a command that should resize the disk by 50Gb.  I have not tested this and did it throught the GUI.
<pre>
qm resize 210 scsi0 +50G
</pre>

