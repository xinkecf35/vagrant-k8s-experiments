# Vagrant Kubernetes Experiments

Just a personal playground to create a local multi-node Kubernetes cluster for personal sandbox for Kuberentes exploration

## Plan

This will use Vagrant to create the multiple VMs with Hyper-V being the hypervisor/VM of choice. Will provision nodes with Ansible running Ubuntu images. Note will be also using WSL since Ansible doesn't really work on Windows.

Will look to create a single control plane node with two workers, mostly dictated by what my personal machine can probably get away with.


## Notes on Hyper-V and rebooting

Rebooting the node will cause the IP address to reset, at least with the default Hyper-V swithc

need to look more into how DNS might work with Vagrant and or understanding how to setup Kubernetes to not mind
the changes in IP