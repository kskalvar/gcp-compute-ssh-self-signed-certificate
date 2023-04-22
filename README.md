Using SSH and a Self-Signed Certificate with GCP Compute Engine  
=======================================================
Abstract:
```
Although the Google Cloud Platform (GCP) provides an easy way to access a Compute Engine Virtual
Machine (VM) through the GCP Console or the GCP SDK gcloud command line, there may
be a use-case in which you would prefer to use ssh.

In this presentation we will show you how to configure a self-signed certificate and upload to
the GCP Compute Service so you can ssh to a Compute VM Instance
```
```
Note: This how-to assumes GCP SDK installed locally and configured to access the GCP Cloud.
```
Steps:  
* [Create cloud_shell](#Create-cloud_shell)
* [Troubleshooting](#Troubleshooting)
* [References](#References)


### Create a ssh self-signed certificate

I've added the associated GCP Service name, date, and userid to the gcp-key name to help identify
which GCP Service it's targeted to as well what account you will required to use to login to the
GCP Compute VM Instance.

```
ssh-keygen -t rsa -f gcp-key-kskalvar-2023-04-22 -C kskalvar -b 2048
```

#### Edit gcp-key-metadata

So we need to edit the metadata a little so it fits the format GCP Compute requires.  There's an
before and after example so you can see the results.

```
cp gcp-key-compute-kskalvar-2023-04-22.pub gcp-key-compute-kskalvar-2023-04-22-pub-metadata

# Vim or sed commands or use any text editor

s/ssh-rsa/kskalvar:ssh-rsa/
s/ kskalvar$//

```
Before:
ssh-rsa <key> kskalvar

After:
kskalvar:ssh-rsa <key>

#### Copy metadata to the GCP Project 

```
gcloud compute project-info add-metadata \
--metadata-from-file=ssh-keys=gcp-key-compute-kskalvar-2023-04-22-pub-metadata \
--project=my-gks-test-project

```
### Create cloud_shell
We'll be using the GCP SDK installed locally to create a GCP Compute VM instance.  You could
easily use the GCP Cloud Console if you're familiar with it.

NOTE: AWS EC2 Key Pair File should be in the directory you run ssh from  
NOTE: You'll need the GCP Compute VM Instance Public IPv4 DNS for your cloud_shell

```
gcloud compute instances create cloud-shell \
--image=ubuntu-1804-bionic-v20230418 \
--image-project=ubuntu-os-cloud \
--machine-type=e2-micro \
--scopes=https://www.googleapis.com/auth/cloud-platform

```
### Connect to cloud_shell and Update Basic Tools

```
gcloud compute instances list
ssh -i gcp-key-compute-kskalvar-2023-04-22 -o "StrictHostKeyChecking no" kskalvar@<EXTERNAL_IP>

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

```
* When ssh'ing to the GCP Compute VM Instance you may receive warnings or errors.  Remove the
existing .ssh directory in your home directory should elimiate these warnings.
```

### References

Connect to Linux VMs
https://cloud.google.com/compute/docs/connect/standard-ssh#openssh-client

Using SSH and a Self-Signed Certificate with GCP Compute Engine
https://github.com/kskalvar/gcp-compute-ssh-self-signed-certificate


