# Jumpstart for NIH Cloud Lab on AWS
---------------------------------

_This repo is in active development. Expect frequent changes until April 28th!_

There are a LOT of resources available to learn more about AWS, which can be overwhelming. NIH Cloud Lab’s goal is to make cloud very easy and accessible for you, so that you can stop wasting time on administrative tasks and focus on your research.

Use this repository to learn about how to use AWS by exploring the linked resources and walking through the tutorials. If you are a beginner, we suggest you begin with this Jumpstart section. If you already have foundational knowledge of AWS and cloud, feel free to skip ahead to the [tutorials](/tutorials/) section for in-depth examples of how to run specific workflows such as genomic variant calling and medical image analyis.

## Overview of Page Contents

+ [Getting Started](#GS)
+ [Command Line Tools](#CLI)
+ [Ingest and Store Data](#STO)
+ [Sagemaker Notebooks](#SAG)
+ [Virtual Machines in EC2](#VM)
+ [Creating a Conda Environment](#CO)
+ [Serverless Functionality](#SER)
+ [Clusters](#CLU)
+ [Billing and Benchmarking](#BB)
+ [Getting Support](#SUP)
+ [Additional Training](#TR)

## **Getting Started** <a name="GS"></a>
All the tutorials you need to learn a lot of what is possible on AWS can be found in the AWS Getting Started [Tutorials Page](https://aws.amazon.com/getting-started/hands-on/?getting-started-all.sort-by=item.additionalFields.sortOrder&getting-started-all.sort-order=asc&awsf.getting-started-category=*all&awsf.getting-started-level=*all&awsf.getting-started-content-type=*all&awsm.page-getting-started-all=2), and we recommend you go there and explore some of the tutorials on offer. Nonetheless, it can be hard to know where to start if you are new to the cloud. To help you, we thought through some of the most common tasks you will encounter doing cloud-enabled research, and gathered tutorials and guides specific to those topics. We hope the following materials are helpful as you explore migrating your research to the cloud.

## **Command Line Tools** <a name="CLI"></a>
Most things in AWS can be done without the command line, but the command line tools will generally make your life easier in the long run. The primary tool you will need is the AWS CLI, which will allow you to interact with instances or S3 buckets from your local terminal. Instructions for the CLI can be found [here](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html). To configure the CLI, you will need to authenticate using access keys. Within Cloud Lab, you will need to use Short Term Access Keys. If you are an intramural user, the instructions for accessing these are found [here](/docs/Intramural_STAKs.md).

If you are running bioinformatic workflows, you will (soon) be able to leverage the serverless functionality of AWS using the [AWS Genomics CLI](https://aws.amazon.com/genomics-cli/), which is kind of a wrapper to Nextflow and Cromwell with Batch. This feature does not yet work within Cloud Lab due to some peculiarities of the environment, so check our [documentation](/docs/agc.md) for periodic updates on access. We are actively working on integrating this tool into Cloud Lab.

## **Ingest and Store Data using Amazon S3** <a name="STO"></a>
Data can be stored in two places on the cloud, either in a cloud storage bucket, which on AWS is called Amazon Simple Storage Service (S3), or on an instance, which usually has Elastic Block Storage. In general, you want to keep your compute and storage separate, so you should aim to storage data in S3 for access, then only copy the data you need to a particular instance to run an analysis, then copy the results back to S3. In addition, the data on an instance is only available when the instance is running, whereas the data in S3 is always available. [Here](https://aws.amazon.com/getting-started/hands-on/backup-files-to-amazon-s3/) is a great tutorial on how to use S3 and is worth going through to learn how it all works. We also wanted to give you a few other tips that may be helpful when it comes to moving and storing data. If your end goal is to move data to an S3 bucket, you can do that using the UI and clicking the `Upload` button, or you can use the CLI by typing `aws s3 cp <FILE> <s3://BUCKET>`. If you want to move a whole folder, then use the --recursive flag: `aws s3 cp <DIR> <s3://BUCKET> --recursive`. The same applies whether moving data from your local directory or from an EC2 instance. Likewise, you can move data from S3 back to your local machine or your EC2 instance with `aws s3 cp <s3://BUCKET/FILE> <DESTINATION/PATH>`. Finally, you can move data to an instance using scp, just make sure the instance is running. You can use a command like `scp -i 'key.pem' <FILE> ec2-user@ec2-NAME.REGION.compute.amazonaws.com:~/PATH `. Once the data is on the VM, it is a good idea to use `aws s3 cp` to move data to S3. If you are trying to move data from SRA to an instance, or to S3, you can use [fasterq_dump](https://github.com/glarue/fasterq_dump) from the SRA toolkit. The best way is probably to install on an EC2 instance, then copy the data to SSD, then optionally copy it to S3 for backup or use elsewhere. 

There is some strategy to managing storage costs as well. When you have spun up a VM, you have already paid for the storage on the VM since you are paying for the size of the disk, whereas S3 storage is charged based on how much data you put in S3. This is something to think about when copying results files back to S3 for example. If they are not files you will need later, then leave them on the VM's EBS and save your money on more important data to put in S3. If the data is important though, either create a disk image as a backup, or copy it to s3, or both! 

## **Launch a SageMaker Notebook** <a name="SAG"></a>
Let's begin with running a SageMaker notebook. Notebooks are ideal for certain problems, particularly when doing a tutorial because you can mix code with instructions. They are also great for exploring your data or workflow one portion at a time, since the code gets broken up into little chunks that you can run one by one, which lends itself very well to most ML/AI problems. The notebook we are going to run is inside this repo, but we are going to launch a Sagemaker instance and then copy the notebook into AWS programatically.

To begin, go to `Services > Machine Learning > Amazon SageMaker`. Once on the landing page, you should see a menu bar on the left side of the page. Click `Notebook` and then `Notebook instances`. Now click the orange colored `Create notebook instance`. Give your instance a globally unique name. Under `Notebook instance type` choose a machine type. You can find the full list of instance types [here](https://aws.amazon.com/ec2/instance-types/). Fortunately not all machines are available with SageMaker, so we only have to choose from a few. You can figure out how much your notebook will cost to run using the pricing calculator [here](https://aws.amazon.com/ec2/pricing/on-demand/) based on your machine type. As long as the notebook is running (and not stopped) you will be charged per second of use. Leave all other values as default for now. Click `Create notebook instance`. It should take about 10 minutes to spin up, so go brew some coffee and come back. Once the status changes from `Pending` to `InService` then you can connect.

### Import our training notebook
Now that our instance is `InService` (read running) we can click `Open JupyterLab` on the far right of the screen. Notice that we have a lot of options for creating new notebooks, and we can also open a terminal window. Let's begin by looking at the AWS example notebooks by clicking the bottom icon on the far left that kind of looks like a brain. You will see that most of these are generic data science and ML topics, but there are a few biomedically relevant examples, with several notebooks focused on cancer data. These notebooks are a great way to learn some basic functionality of AWS like ingesting data, training and running ML/AI models, and running R notebooks. You can also explore a variety of more advanced applications. Open a few notebooks and copy them to your workspace to see how that works. After this, you can copy in a custom notebook and some example data. From the base directory, click the git icon on the middle left bar, it kind of looks like the letter 'T' with a tilt. Click `Clone a Repository`, and then paste the address to this repo (from the green box in the top right) into the box.

Now you have the cloud-lab-training directory available. Navigate to NIHCloudLabAWS > tutorials > notebooks > pangolin > pangolin_pipeline.ipynb.
Explore this notebook and see how data moves in and out of the Sagemaker environment. You can also manually add files, whether notebooks or data using the up arrow in the top left navigation menu. We can easily switch between different kernels in the top right, whether R or Python or Spark. 

Here's a few tips if you are new to notebooks. The navigation menu in the top left controls the control panel that is the equivalent to your directory structure. The panel above the notebook itself controls the notebook options. Most of these are obvious, but a few you will use often are the plus sign to add a cell, the scissors to cut a cell, and the stop to stop a running process. You can use the play button to run a cell, or use shift + enter/return. You can also use CMD + Enter, but it will only run the current cell and not move to the next cell. Above that menu you will see an option called `Kernel` which is useful if you need to reset the kernel, you can click Kernel > Restart Kernel and Clear All Outputs. This will give you a clean restart. You can also use Kernel > Change Kernel if you need to switch between Kernel environments. 

Also worth noting that when you run a cell, sometimes it doesn't produce any output, but things are running in the background. If the brackets next to a cell have an * then it is still running. You can also look at the bottom where the kernel is listed (e.g. Python 3 | status) and it will show either Idle or Busy depending on if anything is running or not. 

## **Spin up a Virtual Machine and run a workflow** <a name="VM"></a>
A lot of great resources exist on how to spin up, connect to, and work on a VM on AWS. The first place to direct you is the tutorial created by [NIH Common Data Fund](https://training.nih-cfde.org/en/latest/Cloud-Platforms/Introduction_to_Amazon_Web_Services/introtoaws3/). This tutorial expects that you will launch an instance and work with it interactively.
[Here](https://aws.amazon.com/getting-started/hands-on/launch-a-virtual-machine/) is an example developed by AWS that gives a good step by step on how to launch and access an instance using Amazon Lightsail. Lightsail is a simplified version of the full AWS console, and may provide an interface you like better for using EC2. Note that resources you spin up in Lightsail will not be available in EC2. A lot of the [tutorials]((/tutorials/)) will have instructions on spinning up EC2 instances as well. 

If you need help on launching a Windows VM, check out this [tutorial](https://aws.amazon.com/getting-started/hands-on/launch-windows-vm/).

From a security perspective, we recommend that you use Center for Internet Security (CIS) Hardened VMs. These have security controls that meet the CIS benchmark for enhanced cloud security. To use these VMs, go to the AWS Marketplace > Discover Products. Then search for `CIS Hardened` and chose the OS that meets your needs. Click, `Continue to Subscribe` in the top right, and then `Continue to Configuration` and set your configuration parameters. Finally, click `Continue to Launch`. Here you decide how to launch the Marketplace solution; we recommend `Launch from EC2`, although you are welcome to experiment with the other options. Now click `Launch` and walk through the usual EC2 launch parameters. Click `Launch` and then you can view the status of your VM in the EC2 Instances page.

## **Creating a Conda Environment** <a name="CO"></a>
Using the conda package manager is one of the easier ways to create a comprehensive compute environment within an instance. Note that the instructions here are for EC2. If you want to create an environment in a Sagemaker Notebook, follow [these instructions](https://github.com/aws/studio-lab-examples/blob/main/custom-environments/custom_environment.ipynb). We recommend using mambaforge since it is a lot faster than the traditional conda, then creating a conda environment with whatever tools you want to use for your particular research aims. Conda environments are created using configuration files in yaml format, where you specify the name of the environment, the conda channels to search, and then the programs to install. You can optionally specify a version for each program, or just list the name and have the default version installed. For example, `- bwa` or `- bwa ==0.7.17` with both install version `0.7.17`, but you could list a different version as needed. Further, some programs you may need do not play well with conda, or are simply not available. If you run into lots of errors while trying to intall something, consider installing via pip or downloading a binary. Make sure if you install anything in addition to the conda environment, you do it after activating the environment. 

To create the conda environment, first install mamba:
```
curl -L -O https://github.com/conda-forge/miniforge/releases/latest/download/Mambaforge-$(uname)-$(uname -m).sh
bash Mambaforge-$(uname)-$(uname -m).sh -b -p $HOME/mambaforge
```
Now add mamba to your path.
```
export PATH="/home/ec2-user/mambaforge/bin:$PATH"
```
List your current conda environments.
```
conda info --envs 
```
Now create the conda environment with all of your desired packages. Note that the file name doesn't matter if you have the name designation at the top of the yaml file. If you don't name the environment in the yaml, then it will be named whatever your file is named. Or you can add a -n flag to name the environment (`mamba env create -f environment.yaml -n your_name_of_choice`).
```
mamba env create -f envs/environment.yaml
```
Now list your environments again to find the path of your new environment.
```
conda info --envs 
```
Now activate your environment using the path printed from the previous command. 
```
conda activate /home/ec2-user/mambaforge/envs/$ENVNAME
```
Now test your environment by running one of the programs you just installed. For example, type `bwa` (if you installed bwa!).

## **Serverless Functionality** <a name="SER"></a>
Serverless is kind of having it's moment. The idea is that you can run things, an analysis, an app, a website, and not have to deal with servers (VMs). There are still servers running under the hood, you just don't have to manage them. All you have to do is call a command that runs your analysis in the background, and copies the output files to a storage bucket. The two AWS serverless features you may want to look into are [AWS Step Functions](https://aws.amazon.com/step-functions/?step-functions.sort-by=item.additionalFields.postDateTime&step-functions.sort-order=desc), which helps build pipelines, and [AWS Batch](https://docs.aws.amazon.com/batch/latest/userguide/what-is-batch.html), which runs your job across multiple VMs similar to submitting a job to a traditional HPC cluster. Here are a few suggestions of how Serverless functionality could be helpful: using a workflow manager like [Snakemake](https://snakemake.readthedocs.io/en/stable/executing/cloud.html), [Nextflow](https://docs.opendata.aws/genomics-workflows/orchestration/nextflow/nextflow-overview.html), [Cromwell](https://docs.opendata.aws/genomics-workflows/) or using the [Amazon Genomics CLI](https://aws.github.io/amazon-genomics-cli/) once it becomes available in Cloud Lab.

## **Clusters** <a name="CLU"></a>
One great thing about the cloud is its ability to scale with demand. When you submit a job to a traditional cluster, you have to specify up front how many CPUs and memory you want to give to your job, and you may over or under utilize these resources. With cloud resources like serverless and clusters  you can leverage a feature called autoscaling, where the compute resources will scale up or down with the demand. This is more efficient and keeps costs down when demand is low, but prevents latency when demand is high (think Black Friday shopping on a website). For most users of Cloud Lab, the best way to leverage scaling is to use AWS Batch, but in some cases, maybe for a whole lab group or large project, it may make sense to spin up a [Kubernetes cluster](https://aws.amazon.com/kubernetes/).

## **Billing, Benchmarking and Cost Controls** <a name="BB"></a>
Many Cloud Lab users are interested in understanding how to estimate the price of a large scale project using a reduced sample size. Generally, you should be able to benchmark with a few representative samples to get an idea of time and cost required for a larger scale project. In terms of cost, the best way to estimate costs is to use the AWS pricing calculator [here](https://aws.amazon.com/ec2/pricing/on-demand/) for an initial figure. Then, you can run some benchmarks and double check that everything is acting as you expect. For example, if you know that your analysis on your local cluster (Biowulf for example) takes 4 hours to run for a single sample with 12 CPUs, and that each sample needs about 30 GB of storage to run a workflow, then you can extrapolate out how much everything may cost using the calculator (e.g. EC2 + S3). To check on costs, go to `Services > Billing`. From there, the `AWS Billing Dashboard` will prove useful, as well as the `Bills` tab. You can drill into each expense and see what each notebook is costing you for example. `AWS Cost Explorer` will probably also prove helpful.

As you go through all the tutorials, you can keep costs down by deleting EC2 instances and S3 data you no longer need. Some workflows like Cromwell on AWS will spin up infrastructure under `Cloud Formation`. Make sure you delete all those resources as well if you don't need them. 

## **Getting Support** <a name="SUP"></a>
You should be added to our Teams channel. Feel free to message others in the group for support and our team will also chime in and help. For NIH Intramural users, you can submit a support ticket to Service Now. For all other users, you can reach out to the STRIDES email with questions at STRIDES@nih.gov. Please be sure for tickets and email to have a clear Subject line, such as AWS help with Nextflow with BATCH. For issues the STRIDES team is unable to remediate, you can reach out to AWS support directly by clicking the question mark in the top right part of the console. 


## **Additional Training** <a name="TR"></a>
If you are interested in additional cloud training opportunities please visit the [STRIDES Training page](cloud.nih.gov/training/). For more information on the STRIDES Initiative at the NIH, visit [our website](cloud.nih.gov) or contact the NIH STRIDES team at STRIDES@nih.gov for more information.

