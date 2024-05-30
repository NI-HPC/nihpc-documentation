# Quick-Start Guide

## Login nodes

After connecting to Kelvin2 you will start your session in one of four login nodes as indicated in the terminal window.

```bash
[<username>@login1 [kelvin2] ~]$
```

While using a login node, you are only permitted to do small tasks such as

- managing files
- small data transfers
- creating, editing and submitting jobscripts
- checking and managing jobs


!!! warning

    Please do not run computationally expensive jobs on a login node. There are only 4 login nodes split between all Kelvin2 users and running high workloads here will seriously impact the other users.

## Running your jobs on compute nodes
A user wishing to run a job on a compute node must first write a *batch script*. This batch script contains specifications for the required computational resources, instructions for loading program dependencies, and commands to execute the code. Once the batch script is complete, this is submitted to a job scheduler called [SLURM](https://slurm.schedmd.com/documentation.html){target=_blank} and the job will subsequently be queued and run on a compute node when resources become available. The scheduler ensures that all users get their fair share of system resources. 

This process is quite different to how you normally run programs on your own personal computer. As such, the following step-by-step guide explains how to run a simple Python 'Hello World' job on a compute node using a batch script submitted to the job scheduler. This walkthrough is designed to be very minimal to provide a first-time user with an overview of the process. More information is contained in the Running Jobs documentation page.

### Step-by-step guide to submitting your first job on Kelvin2

Click the following steps to expand:

??? info "1. Create Python executable file"

    - Enter the following command into your Kelvin2 terminal to create the file for our 'Hello World' program
        ```bash
        vi hello-world.py
        ```
    - Copy the following lines of Python code into the file, then save and exit vim. 
        ```python
        import os

        print("hello world! I am running on", os.environ['HOSTNAME'])
        ```
        This code will print a hello world message and the name of the compute node the code is running on to the standard output stream. If you were running this program directly, this would be printed to your terminal.
 
??? info "2. Create SLURM batch script"

    - Enter the following command into your Kelvin2 terminal to create the batch script file that we will use to submit `hello-world.py` to the job scheduler.
        ```bash
        vi hello-world-jobscript.sh
        ```
    - Copy the following lines of code into the file, then save and exit vim. 
        ```bash
        #!/bin/bash

        #SBATCH --output=hello-world-job.output
        #SBATCH --time=00:00:10

        module load apps/python3/3.10.5/gcc-9.3.0

        python3 hello-world.py
        ```
      In this example, the batch script simply specifies:

          - the file in which the program output will be written
          - a time limit after which the job will be automatically closed if it has not yet completed
          - which Python module should be loaded to run the code
          - a line which uses a bash command to execute our Python program

??? info "3. Submit the batch script"

    - Enter the following command into your Kelvin2 terminal to submit the batch script to the scheduler. The job will then be queued and ran on a compute node whenever resources become available.
      ```bash
      [<username>@login1 [kelvin2] ~]$sbatch hello-world-jobscript.sh
      Submitted batch job <jobid>
      ```


??? info "4. Check Status of Job"

    - Enter the following command into your Kelvin2 terminal to check the status of your submitted jobs.
      ```bash
      [<username>@login1 [kelvin2] ~]$squeue -u <username>
      ```
    - If you were quick typing the above command, your may see your job status as Running (R) or Pending (PD). If you do not see your job in your queue it means your job is already complete. 

??? info "5. View output of job"

    - After the job is complete, enter the following command into your Kelvin2 terminal to view the output of your program.
      ```bash
        [<username>@login1 [kelvin2] ~]$ cat hello-world-job.output
        hello world! I am running on node167
      ```
    - You have now successfully ran a job on a compute node.



## Storing your files on Kelvin2

### Home Directory

Your home directory is found at `/users/$USER`. If you followed the step-by-step guide above, then this is where you have stored `hello-world.py`, `hello-world-jobscript.sh` and `hello-world-job.output`.

By default, the contents of your Home directory are private. You can check permissions using the `ls` command.

```bash
[<username>@login1 [kelvin2] ~]$ ls -ald $HOME
drwx------ 17 <username> clusterusers 4096 Oct 31 10:12 /users/<username>
```

Each user has a quota for how much data (50GB) and how many files (100k) they can store in their home directory. The limit on the number of files appears very large, however some user installations can generate lots of small files (e.g. Anaconda) which can breach this limit.  The quota can be checked using the `quota -s` command as follows:
```bash
[<username>@login1 [kelvin2] ~]$ quota -s
Disk quotas for user <username> (uid <userid>):
     Filesystem   space   quota   limit   grace   files   quota   limit   grace
storage1:/export/users
                 36512K  51200M  76800M             292    100k    150k
```

In this case, the user is using 36512KB out of their 50GB disk space quota and has 292 files out of their 100k file limit quota. 

### Shared Scratch Directory

Users also have access to a Shared Scratch directory `/mnt/scratch2/users/$USER`.

The Shared Scratch directory does not have a defined quota for each user and is therefore more suitable for storing larger amounts of data. However, the Shared Scratch directory has a total capacity of 2PB and therefore any data that has not been accessed in the previous 90 days is subject to deletion. 

By default, the Shared Scratch directory is more open than your Home directory. You can check the permissions using the `ls` command as follows:

```bash
[<username>@login1 [kelvin2] ~]$ ls -ald /mnt/scratch2/users/$USER
drwxr-x--- 17 <username> clusterusers 4096 Sep 21 13:52 /mnt/scratch2/users/<username>
```

This Shared Scratch directory is typically used for

  - sharing data between cluster users
  - temporary storage of large amounts of data (either as input to or output from jobs)
  - user installations that generate a large number of small files (e.g. using Anaconda)


### Temporary Directory

Each compute node has a local scratch directory `/tmp` for storing data during a job. Although using this storage may result in faster read/write times during a job, this data will automatically be deleted when the job completes. 

The temporary directory is typically used for 

  - temporary files produced during a job that are not required after job completion
  - files that need to be read/written to multiple times within a job

### McClay Research Data Storage

QUB PIs may also request access to the McClay Research Data Storage (McClayRDS) for the duration of their research project. The storage space will be assigned to individual research projects and access is shared amongst members of that research group. Access can be requested by filling in the application form [here](https://forms.office.com/pages/responsepage.aspx?id=6ner6qW040mh6NbdI6HyhnnikCeespJIiWT3GPl56KZUNDUyNUdLT1BXV0RROU03MkVHT1ZVNEZZRS4u&origin=lprLink){target=_blank}. Once assigned a project code, the directory is accessible at  `/mnt/autofs/mcclayrds-projects/<project-code>`. 

McClayRDS is typically used for

- storage of raw data that is actively being used during a research project
- data that is required to be stored in a secure environment, replicated to a second site.

!!! note

    Data in the McClayRDS is replicated to a second site to protect against data loss from disk failure/damage. The data sync occurs every 20 mins. This data is not backed up, so data loss from user modification or deletion will not be recoverable.


### Storage Summary

|Description|Directory|Disk Space Limit| File Number Limit|File Deletion Policy|
|---|---|---|---|---|
|Home|`/users/$USER`|50GB|100k|No automatic file deletion|
|Scratch|`/mnt/scratch2/users/$USER`|N/A|N/A|Deletion after 90 days without access|
|Temp|`/tmp`|Disk Space on Node|N/A|Deletion after session ends|
|McClayRDS|`/mnt/autofs/mcclayrds-projects/<project-code>`|Project specific|N/A|Deletion after specified data retention period ends|


## Transferring Data with Kelvin2

### Transfer files using graphical interfaces
Files can be simply transferred to and from Kelvin2 using a 'drag and drop' interface with a graphical file browser such as [MobaXterm](https://mobaxterm.mobatek.net){target=_blank} or [WinSCP](https://winscp.net/eng/index.php){target=_blank}.

### Transfer small files and folders using `scp`

Individual files can be transferred between your local computer and Kelvin2 by entering the following commands into your local terminal:

**Uploading a file**

From Inside the QUB Network:
```bash
scp path/to/local/file.txt <username>@kelvin2.qub.ac.uk:/path/on/kelvin2
```

From Outside the QUB Network:
```bash
scp -P 55890 -i /path/to/ssh/private/key path/to/local/file.txt <username>@login.kelvin.alces.network:/path/on/kelvin2
```

**Downloading a file**

From Inside the QUB Network:
```bash
scp <username>@kelvin2.qub.ac.uk:/path/to/kelvin2/file.txt path/to/local/directory/
```

From Outside the QUB Network:
```bash
scp -P 55890 -i /path/to/ssh/private/key <username>@login.kelvin.alces.network:/path/to/kelvin2/file.txt path/to/local/directory/
```


Directories and their contents can also be transferred by specifing a directory instead of a file and using `scp` with a `-r` flag (for recursive).

!!! note

    When using `scp`, note that the `-P` flag to specify the port is uppercase. This is a frequent cause of confusion since the flag to specify the port when using the `ssh` command is lowercase.


### Transfer large amounts of data

For large amounts of data, instead of using the login nodes as shown above, please use one of the data movement nodes as this will be faster and cause less disruption to other users. The host names of these are given below:

- dm1.kelvin.alces.network
- dm2.kelvin.alces.network

!!! note

    If you are transferring a large number of small files, it is recommended to first pack the files into an "archive" file using tools like zip or tar. For large data transfers it may also be worth looking into alternative methods such as `rsync` which allow file transfers to be stopped and restarted if required. For further support with large date transfers, please [contact us](https://www.ni-hpc.ac.uk/contact/){target=_blank}.
