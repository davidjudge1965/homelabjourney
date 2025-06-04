# Home Lab Journey
This repo holds my notes, instructions and explanations or thoughts around my journey into homelabbing.

I've been homelabbing for many years now, but as I embark on new explorations I wanted to document the journey.

I have implemented or plan to implement k8s (in the form of [k3s](https://k3s.io)), implement some monitoring (probably Prometheus and Grafana), and create some AI agents (probably using [n8n](https://n8n.io/)). Some app may be deployed to k3s along the way.

Underlying this effort are:
- One Proxmox VE server (AMD Ryxen 5 5600G, 64GiB RAM, 5GiB Storage)
- Docker VM (for earlier experiments)
- Ansible server VM
- My deskside computer (Ryzen 5 5600, 48GiB) can run VMs too:
    - Hyper-V 
    - VMware Workstation Pro


## A bit about me
I've been working in IT for all my professional life and for some time before that as a student - my first job (coding) was at age 14.

Since the early 1980's, I have been employed as:
- Support engineer / Escalations Manager
- IT Manager
- Consultant (in Business Rules)
- Presales in AIOps, Observability, NetOps, Automation, etc.

If you want to find out more about my career, check out [my LinkedIn page](https://www.linkedin.com/in/davidjudge) or my CV which [can be downloaded from here](https://resume.davidmjudge.me.uk).


## Setting up the homelab
Key steps/processes in setting-up my homelab, though these may not be accomplished in the order below, are:
- Install hardware and operating systems (not documented)
- Build the basics of VM creation through the CLI, ready for a future automation or orchestration
- Set up Kubernetes (not documented - there's already enough on the internet explaining this)
- Set up monitoring of k3s, proxmox, etc. along with some dashboarding
- Set up a front page to make access to the various applications easy
- etc.

In the next section you will find links to the various articles I have written.
You will probably find the articles quite wordy.  One of my frustrations with many of the articles, blogs amd videos about how to do stuff tell you the step, but mostly don't really try to give a rationale.  I have tried to give the context and rationale or explanation for most of the steps I have taken and documented.

## Blogs/Articles

[Creating the VMs with CloudInit](articles/proxmox-cloudinit-CLI.md)