Using SSH and a Self-Signed Certificate with GCP Compute Engine  
=======================================================
Abstract:
```
Google Cloud Platform (GCP) provides an easy way to access a Compute Engine Virtual
Machine (VM) through the GCP Console or the GCP SDK gcloud command line, there may
be a use-case in which you would prefer to use ssh.

In this presentation we will show you how to configure a self-signed certificate and upload the
certificate metadata to the GCP Compute Service so you can ssh to a Compute VM Instance from your
local machine.
```
```
Note: Assumes you have a GCP Account, you have a GCP Project created and Compute Engine access
      enabled, and you have the GCP SDK installed locally and configured to access the GCP Cloud  

```
Steps:  
* [Check GCP Compute Engine API Enabled](#Check-GCP-compute-Engine-API-Enabled)
* [Create a ssh self signed certificate](#Create-a-ssh-self-signed-certificate)
* [Create cloud shell](#Create-cloud-shell)
* [Connect to cloud shell and Update Basic Tools](#Connect-to-cloud-shell-and-Update-Basic-Tools)
* [Delete cloud shell](#Delete-cloud-shell)
* [Troubleshooting](#Troubleshooting)
* [References](#References)

### Check GCP Compute Engine API Enabled

Be sure you have the GCP Compute Engine API enabled so you can create your VM Instance.
Recommend you use the GCP Console.  But it will prompt you when you create the VM Instance using
the GCP SDK if it isn't already enabled.

```
gcloud services list --enabled | grep Compute
```

### Create a ssh self signed certificate

I've added the associated GCP Service name, date, and userid to the gcp-key name to help identify
which GCP Service it's targeted to as well what account you will be required to use to login to the
GCP Compute VM Instance.

```
ssh-keygen -t rsa -f gcp-key-compute-kskalvar -C kskalvar -b 2048
```

#### Edit gcp-key-metadata

So we need to create a metadata file that fits the format GCP Compute Settings/Metadata requires.
There's an before and after example below so you can see the difference.

```
cp gcp-key-compute-kskalvar.pub gcp-key-compute-kskalvar-pub-metadata
```
##### vim or sed commands to use or use any text editor
```
s/ssh-rsa/kskalvar:ssh-rsa/
```
```
s/ kskalvar$//
```

##### What your metadata should look like
Before:  
ssh-rsa \<key\> kskalvar

After:  
kskalvar:ssh-rsa \<key\>

#### Copy the metadata to the GCP Project Compute Settings 

We'll use the GCP SDK to upload the metadata to the GCP Compute Service, but you can also use
the GCP Console to do this as well. See: GCP Compute Engine/Settings/Metadata/SSH KEYS section

```
gcloud compute project-info add-metadata \
--metadata-from-file=ssh-keys=gcp-key-compute-kskalvar-pub-metadata
```
### Create cloud shell

We'll be using the GCP SDK installed locally to create a GCP Compute VM instance.  You could
easily use the GCP Cloud Console if you're familiar with it.

```
gcloud compute instances create cloud-shell \
--image=ubuntu-1804-bionic-v20230510 \
--image-project=ubuntu-os-cloud \
--machine-type=e2-micro \
--scopes=https://www.googleapis.com/auth/cloud-platform
```
### Connect to cloud shell and Update Basic Tools

NOTE: The GCP Private Key you created should be in the directory you run ssh from  
NOTE: You'll need the GCP Compute VM Instance Public IPv4 DNS for your cloud_shell  
NOTE: The userid you use to create the self-signed certificate is the userid you will use for login

```
ssh -i gcp-key-compute-kskalvar -o "StrictHostKeyChecking no" kskalvar@<EXTERNAL_IP>
```
Install Basic Tools
```
sudo apt update
```
### Delete cloud shell

Using the GCP SDK installed locally to delete the GCP Compute VM Instance we used as the cloud_shell.
```
gcloud compute instances delete cloud-shell --quiet
```
### Troubleshooting

* When ssh'ing to the GCP Compute VM Instance you may receive warnings.  Remove the
existing .ssh directory in your home directory should elimiate these warnings.

* Invalid ssh key entry - unrecognized format: ssh-rsa.  When viewing the cloud-shell VM logs you
will notice an error for an "Invalid ssh key entry".  This error will not keep you from logging into
the VM. See GCP Console/Compute Engine/VM Instances/cloud-shell/three dots on the right/View logs

* When running gcloud compute instances create cloud-shell with --image=ubuntu-1804-bionic-v20230510
you will received notice the image is outdated.  It will give you a recommended image.  Use that one
instead.

### References

Connect to Linux VMs  
https://cloud.google.com/compute/docs/connect/standard-ssh#openssh-client

Using SSH and a Self-Signed Certificate with GCP Compute Engine  
https://github.com/kskalvar/gcp-compute-ssh-self-signed-certificate

Using SSH and a Self-Signed Certificate with GCP Compute Engine Presentation  
https://www.youtube.com/@kal_technology/videos


