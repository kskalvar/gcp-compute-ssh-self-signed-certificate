Using SSH and a Self-Signed Certificate with GCP Compute Engine  
=======================================================
Abstract:
```
Although the Google Cloud Platform (GCP) provides an easy way to access Compute Engine Virtual
Machines (VM) either through the GCP Console or the GCP SDK gcloud command line, there may
be a use-case in which you would prefer to use ssh.

In this presentation we will show you how to configure ssh to login to a Compute VM Instance
```
This solution shows how to create a Self-Signed Certificate and use ssh to login to a GCP Compute
Virtual Machine
```
Note: This how-to assumes GCP SDK installed locally and configured to access the GCP Cloud.
```
Steps:  
* [Create cloud_shell](#Create-cloud_shell)
* [Troubleshooting](#Troubleshooting)
* [References](#References)


### Create ssh self-signed certificate

ssh-keygen -t rsa -f gcp-key -C kskalvar -b 2048
cp gcp-key.pub gcp-key-metadata

### Edit gcp-key-metadata
s/ssh-rsa/kskalvar:ssh-rsa/
s/ kskalvar$//

### Copy metadata to the GCP Project 

gcloud compute project-info add-metadata --metadata-from-file=ssh-keys=gcp-key-metadata --project=my-gks-test-project

### Create cloud_shell
We'll using the GCP SDK installed locally to create a GCP Compute VM instance.

### Connect to cloud_shell and Install Basic Tools

NOTE: AWS EC2 Key Pair File should be in the directory you run ssh from  
NOTE: You'll need AWS EC2 Public IPv4 DNS for your cloud_shell

```
gcloud compute instances create cloud-shell \
--image=ubuntu-1804-bionic-v20230131 --image-project=ubuntu-os-cloud \
--machine-type=e2-micro \
--scopes=https://www.googleapis.com/auth/cloud-platform

```

```
ssh -i <AWS EC2 Key Pair File> ubuntu@<Public IPv4 DNS> -o ExitOnForwardFailure=yes

```
Install Basic Tools
```
sudo apt update

```
### Delete cloud_shell
Using the GCP SDK installed locally to delete the GCP Compute VM Instance we used as the cloud_shell.
```
gcloud compute instances delete kubeflow-cloud-shell --quiet  
```
### Troubleshooting

### References



