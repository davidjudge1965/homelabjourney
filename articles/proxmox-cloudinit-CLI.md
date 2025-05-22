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
6. Convert the VMto a template
7. Configure CloudInit (the parameters, that is) 
8. Clone the template into a new VM
9. Update the CloudInit configuration in the VM to meet your needs


### Download the CloudInit image
Canonical's repository of Cloud Init images is [here](https://cloud-images.ubuntu.com/)
You will need to download the image that suits you needs.  For me that was:
https://cloud-images.ubuntu.com/noble/current/noble-server-cloudimg-amd64.img

To perform the download, use this command (replacing the URL with your choice of CloudInit image) while logged into your proxmox server as root.  

```bash
wget https://cloud-images.ubuntu.com/noble/current/noble-server-cloudimg-amd64.img
```

### Create the new VM (to later be converted to a template)
Pick the values you need for memory, CPU, etc.
```bash
qm create 8200 --memory 4096 --core 4 --name ubuntu-cloud --net0 virtio,bridge=vmbr0
```
