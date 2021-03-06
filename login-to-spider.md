# Logging onto Spider

In this section you will login to the platform and familiarise yourself with your working environment:

1. [Login to Spider](#spider-login)
2. [Get familiar with the login node](#spider-env)
3. [Submit your first job](#job-submit)

### <a name="spider-login"></a> 1. Login to Spider

The login credentials (e.g., spidercourse-g01) will be provided to you at the start of the session. Open a terminal in your laptop 
    
 ```sh
 ssh username@spider.surfsara.nl #replace `username` with the username assigned to you
 ```
  
### <a name="spider-env"></a> 2. Get familiar with the login node

Familiarize yourself with your environment:

 ```sh
 whoami
 id $USER
 pwd
 sinfo
```

> **_Food for brain:_**
>
> * What does the output of the command "id $USER" tell you?
> * How many worker nodes are available on the cluster?
> * How many cores per worker node are available on the cluster? Hine: Try the command "scontrol show node"


### <a name="job-submit"></a> 3. Submit your first job

Let us download our first job script and inspect it:
  
 ```sh
 wget https://raw.githubusercontent.com/sara-nl/spidercourse/master/scripts/my-first-job.sh

 cat my-first-job.sh
 ```
 The #SBATCH flags that you see in the script have the following function:
 
 -t: max total run time of the job, here it is 10 minutes  
 -c: 1 core requested   
 --constraint: here we request nodes with skylake processors and ssd local scratch disk space
 
Now that you have inspected the script that will submit your job, let's submit it running the following command:
  
 ```sh
 sbatch my-first-job.sh  #This command will submit a job and give you a unique jobID in return
 squeue -u $USER  #Check the status of your job
 ls  #Check if the output, slurm-jobID.out, is present
 cat slurm-jobID.out #Replace jobID with the ID of your job
 
 #If you wish to cancel a running job you can do so with the following command
 scancel 12345 #Replace 12345 with your corresponding jobID
 ```
 
Let us move to the [next](https://github.com/sara-nl/spidercourse/blob/master/demo-spider-roles.md) section where you will explore the Spider platform further.
