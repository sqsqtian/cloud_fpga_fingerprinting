# Fingerprinting Cloud FPGA Infrastructures

This folder contains the open-source code for the paper
"[Fingerprinting Cloud FPGA Infrastructures](https://doi.org/10.1145/3373087.3375322)".
The software provided will extract fingerprints from the DRAM modules attached to each FPGA slot of AWS F1 instances.

This code can be downloaded from [CASLAB code page](https://caslab.csl.yale.edu/code/cloud-fpga-fingerprinting/).

Prerequisites:
  - Access to [AWS F1](https://aws.amazon.com/ec2/instance-types/f1/) instances
  - A computer with an SSH client

## Setup

### AWS Account

This guide assumes that you have an AWS account and a cryptographic key pair, and are familiar with launching AWS EC2 instances.
If not, please also consult [this guide](https://docs.aws.amazon.com/quickstarts/latest/vmlaunch/step-1-launch-instance.html).

### Access to AGFIs

This code requires access to pre-compiled bitstreams (AGFIs) that can be loaded onto the F1 instance (to-be-created using following instructions). The AGFIs are public and can be accessed (read) by anybody in *us-east-1* region.
Please see the `scripts_running/experiment.py` for the specific AGFI numbers.

### Create F1 Instance

As Amazon EC2 F1 compute instances are not available in all regions, please ensure you are launching an F1 instance from an appropriate region, such as *us-east-1*. You may need request a "limit increase" for F1 instances from AWS support if none are available to you.

When creating the F1 there are two important parameters to choose correctly:
1. The Amazon Machine Image (AMI): To choose the appropriate AMI, search for "FPGA" in "AWS Marketplace", and click **Previous versions** (not "Select") of *"FPGA Developer AMI"*. Then click "Continue to Configuration".
After that, choose "1.6.0" in "Software Version".
2. The instance type: There are three types of F1 instances on AWS: *f1.2xlarge*, *f1.4xlarge*, and *f1.16xlarge*, with 1, 2, and 8 FPGAs attached respectively. All three can be used for the experiments, but this guide uses **f1.2xlarge**.

When the state of the instance is "running", you can click "Connect" and ssh into it:
```sh
ssh -i "YourKey.pem" centos@ec2-XXX.amazonaws.com
```
Please note that the user name is *centos* rather than *root*.

### Instance Configuration

The AWS CLI (Command Line Interface) tools need to be configured with your security credentials (not account password), which can be created on [security credentials](https://console.aws.amazon.com/iam/home?#/security_credential) page.
To do so, you may need to select *Continue to Security Credentials* in a pop-up window.
Expand the *Access keys (access key ID and secret access key)* tab. Click *Create New Access Key*.
Make sure to download the access key file, as you will not have access to the secret key after you finish the access key creation process.

AWS CLI can then be configured using the `aws configure` command from inside the F1 instance. You will need to enter your access key and the corresponding secret key. Also, make sure to set the region to be the same to your F1 instance (e.g. **us-east-1**), and select the output format as **json**.
Note that the input text is case-sensitive, i.e., *us-east-1* is not the same as *US-East-1*. You can re-run the command if needed to replace your answers.

## Runing Experiments

First, clone a copy of AWS's `aws-fpga` repository in the home directory of the VM using the command
```sh
git clone https://github.com/aws/aws-fpga
```
Then, setup the Software Development Kit (SDK):
```sh
cd ~/aws-fpga/
source sdk_setup.sh
```
You may need to source the SDK scripts each time you log into the F1 VM.

### Data Collection

Copy [scripts_running](scripts_running/) folder from this archive to your VM.
Go to the *scripts_running* path inside the VM, and then run the following command:
```sh
make experiment
```
Your will get a folder starting with the instance ID "i-XXX", which contains DRAM PUFs collected from the 4 DRAMs attached to the FPGA (*f1.2xlarge*).

### Data Analysis

To reduce the costs of running the experiment, download the data to your local machine and *stop* the F1 instance.
Then use the scripts inside the [data_processing](data_processing/) folder from this archive to analyze the raw data, and plot the PUF output:
```sh
python plot_bitmap.py -d YourPath/i-XXX_XXX_slot0_XXX/XXs_XXX/X_XXX.dat
```

# Further Information

For more information about our group's research, please visit [CASLAB](https://caslab.csl.yale.edu/publications.html) web page.

