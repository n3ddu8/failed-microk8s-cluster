# PiCluster

This project is intended as a fun way for me to learn new skills. It’s not designed to be production-ready, nor should it necessarily be viewed as an example of best practices.

I’m blogging about my work on this project to share my thought process and explain the choices I’ve made along the way.

## The Aim
Create a cluster that serves all my computing needs, such that the only other computing devices I require are my phone and tablet.

## Hardware
x3 Raspberry Pi 4b 8GB
x1 Raspberry Pi 4b 4GB
x4 Raspberry Pi POE+ Hats
x3 Crucial 250GB SSDs
x1 TP-Link TL-SG1210P 10-Port Gigabit Desktop Switch

## Software
Ubuntu 24.04 LTS
MicroK8s

## Motivation

I already owned several Raspberry Pi devices from previous projects and frequently use a tablet for content consumption. One day, the thought struck me: What if I could reduce my digital possessions to just these devices—selling my desktop PC, laptop, server, NAS, etc.?

As the saying goes:

“Minimalism is not about the absence of possessions, but about focusing on what truly brings value to your life.”

### Why?
•	It’s a fun project
•	It’s a chance to learn
•	Decluttering feels great
•	Selling unused devices makes some cash
•	It reduces energy consumption

## The First Challenge: Build a Cluster
Currently, my 4GB Pi is running Home Assistant. For now, I’m focusing on building a cluster with the three 8GB Pi devices, with plans to expand later.

Since the cluster is running on Raspberry Pi devices, the implementation needed to be lightweight. I also wanted to automate as much of the setup process as possible to make it reproducible and easy to expand.

A few months ago, I started working on Blueberry, an Ansible project to automate the deployment of applications running in Podman on Fedora IoT. It was my first foray into writing Ansible playbooks, and I really enjoyed the experience. Initially, I planned to expand this project, replacing Podman with k3s. However, while researching lightweight Ceph implementations (something I’ve wanted to experiment with), I discovered MicroCeph by Canonical. This led me to MicroK8s.

The combination of MicroCeph and MicroK8s seemed like the perfect fit—they integrate seamlessly and have extensive documentation for running on Raspberry Pi clusters.

Switching to this implementation also meant an operating system change. MicroK8s and MicroCeph are distributed as snaps, which don’t work well with atomic OSes like Fedora IoT. 

As I would be losing the rebasing functionality inherent in Atomic OSes, I wanted something stable. Debian as the official OS of the Raspberry Pi would have been a great choice, however, I opted for Ubuntu as it’s likely to be the most compatible with  MicroK8s and MicroCeph (given they are all developed by Canonical). Thus, I imaged my three SD cards with Ubuntu 24.04 LTS.

At this stage, I’m still using my Linux desktop for developing and running the Ansible playbook.

### Setting Up the Cluster

I began by adding my SSH key to each Pi with the command:
`ssh-copy-id <piuser@pi-ip>`

Next, I created an inventory file and added my three Pi devices to it. This allowed me to test connectivity by running:
`ansible edge -i inventory.yaml -m ping`

When all devices responded with a pong, I moved on to creating the playbook.

### The Playbook

The first step was straightforward. Using the documentation, I created a playbook to install and initialize MicroK8s on each Pi.

Adding the nodes to the cluster was significantly more challenging however. To add the two devices (pibeta and pigamma) to the cluster controlled by pialpha, the following steps are required:
	1.	Run microk8s add-node on pialpha.
	2.	Copy the generated command (with its token) to the appropriate device.
	3.	Execute the command on pibeta.
	4.	Repeat the process for pigamma.

A simple regex sufficed to parse the command from the output, but the challenge was Ansible’s behavior. It executes tasks on all hosts in parallel, so the token for pibeta would be overwritten by the token for pigamma before ever being executed on pibeta.

Eventually I solved this by offloading the sequential tasks to a separate playbook and using delegate_to to ensure the correct command was executed on the correct Pi at the appropriate time.

Once the nodes were successfully added, I refactored the playbook to dynamically determine which nodes in the inventory file needed to be added to the cluster, ensure idempotency, so additional nodes could be set up and added later without issues and reduce redundant code.

With these steps complete, I now have a lightweight, automated process for building and expanding my Raspberry Pi cluster.

My Pi’s at the end of day 1:
![Day 1](/images/day1pi.JPEG)

