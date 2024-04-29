# Running Jobs

On Kelvin2, jobs are submitted via *batch script* to a  scheduling system called [SLURM](https://slurm.schedmd.com/documentation.html){target=_blank}. Jobs are not necessarily run in the order in which they are submitted and those jobs which require a large number of cores, memory or walltime will have to queue until the requested resources become available. The system will run smaller jobs, that can fit in available gaps, until all of the resources that have been requested for the larger job become available.

!!! note

    To improve your experience with the queue, and that of other users, please ensure that you are not requesting excessive amounts of resources to complete your jobs. After reading the below documentation, see our [tips for optimising your jobscript](#tips-for-optimising-your-jobscript).

## Batch Scripts
A SLURM batch script typically contains four main components

- A 'shebang' `#!` which states the interpreter type, i.e. `#!/bin/bash` 
- A series of `#SBATCH` directives that specify resource requirements of the job and a variety of other attributes
- Lines that load modules and set the environment
- At least one executable bash line that runs code and any associated input parameters

An brief overview of `#SBATCH` directives and modules on Kelvin2 is given below:

### `#SBATCH` directives

`#SBATCH` directives specify the resources required for your job and various other attributes. The definitive list of these are in the [SLURM documentation](https://slurm.schedmd.com/sbatch.html){target=_blank}, and a few of the most commonly used ones are given below.


**Attribute Directives: e.g. `--job-name`, `--output`, `--error`, `--time`, `--mail-type`, `--mail-user`**


`--job-name` - Specifies a name for the job. This helps identify the job when querying your queues.

`--output` - Specifies the file in which the standard output will be written. This is the output that would have been printed to the terminal, if you were running the application directly.

`--error` - Specifies the file in which the standard errors will be written.

`--time` - Specifies a time limit for the job after which it will automatically exit, e.g. 01:30:00 for 1.5 hours, or 1-00:00:00 for 1 day.  Providing an accurate time limit will help your queued jobs start running more quickly.

`--mail-type` - Notifies the user by email when various events occur, like when the job starts or ends.

`--mail-user` - Specifies the email address to receive the notifications.

```bash
#SBATCH --job-name=my-kelvin2-job
#SBATCH --output=my-output-file
#SBATCH --error=my-error-file
#SBATCH --time=01:30:00
#SBATCH --mail-type=ALL
#SBATCH --mail-user=<user-email-address>
```

**Partition Directive: `--partition`**

The `--partition` directive should be specified according to the resource requirements and time limit of your job. For example, if you wanted to run a job with a wall time of 3 days, you would need to use a low priority partition otherwise your job would never run, e.g.

```bash
#SBATCH --partition=k2-lowpri
#SBATCH --time=3-00:00:00
```

The main Kelvin2 partitions along with their time limits and computational resources are shown below. The default partition, if none is specified in the batch script is k2-hipri.

|Partition|Time Limit|CPU Cores per Node|Memory (GB) per Node |
|-|-|-|-|
|k2-hipri|3 hours|128 (94 Nodes)|773 GB (59 Nodes), 1023 GB (35 Nodes)|
|k2-medpri|24 hours|"|"|
|k2-lowpri|N/A|"|"|
|k2-epsrc|N/A|"|"|
|k2-himem|3 days|128 (6 Nodes), 256 (2 Nodes)|2051 GB (4 Nodes) 2063 GB (4 Nodes)|
|k2-epsrc-himem|N/A|"|"|
|k2-gpu|3 days|48 (8 Nodes), 128 (4 Nodes)|514 GB (8 Nodes), 1031GB (4 Nodes)|
|k2-gpu-interactive|3 hours|"|"|
|k2-epsrc-gpu|3 days|"|"|

Other partitions are also available, including some for specific research groups. A comprehensive list of Kelvin2 nodes, their associated partitions and their computational resources can be found using the [sinfo command](https://slurm.schedmd.com/sinfo.html){target=_blank}.


**Resource Directives: e.g. `--ntasks`, `--nodes`, `--cpus-per-task`, `--mem-per-cpu`**

`#SBATCH` directives for computational resources follow the format

```bash
#SBATCH --<resource-type>=<amount>
```


*MPI Application example*

If your MPI application requires 40 CPU cores spread across a maximum of 2 nodes and 80GB memory (2GB per CPU core) you would include the directives

```bash
#SBATCH --ntasks=40
#SBATCH --mem-per-cpu=2G
#SBATCH --nodes=2
```

*OpenMP Application example*

 If your OpenMP application require 40 CPU cores all on the same node and 80GB memory (2GB per CPU core) you would include the directives

```bash
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=40
#SBATCH --mem-per-cpu=2G
#SBATCH --nodes=1
```

**GPU Directives: `--gres`**

Jobs submitted to the k2-gpu, k2-gpu-interactive (for [interactive jobs](#interactive-jobs) only) and k2-epsrc-gpu partitions can access GPUs using the `--gres` flag by following the below template (`<N>` is the number of GPUs requested per node)

```bash
#SBATCH --gres=gpu:<gpu-type>:<N>
```

|Available GPU Resources|Example `--gres` flag|
|-|-|
|32 x Tesla V100 PCIe 32Gb (4 GPUs on 8 Nodes)|`#SBATCH --gres=gpu:v100:1`|
|12 x Tesla A100 SXM4 80GB (4 GPUs on 3 Nodes)|`#SBATCH --gres=gpu:a100:1`|
|28 x CI slices of a Tesla A100 SXM4 80GB (7 slices on 4 GPUs on 1 Node)| `#SBATCH --gres=gpu:1g.10gb:1`|




### Loading Modules

Kelvin2 has a large repository of software installed directly on the system which can be loaded for the jobs using the batch script.

The `module avail` command will show the list of applications that are currently installed on the central repository.

```bash
[<username>@login1 [kelvin2] ~]$ module avail
---  /opt/gridware/local/el7/etc/modules  ---
  apps/anaconda/2.5.0/bin
  apps/anaconda3/2021.05/bin
  apps/anaconda3/2022.10/bin
  apps/anaconda3/5.2.0/bin
  apps/annovar/20160201/noarch
  apps/bamtools/2.3.0/gcc-4.8.5
  ...
```

Press `space` to navigate through the list one page at a time and when done press `q` to exit.

The most useful commands related to working with modules are:

- `module display <module>` -  Shows information about the software
- `module load <module>` - load the selected module, including its binaries/libraries into the user's environment
- `module unload <module>` - removes the module from the user's environment
- `module list` - show the currently loaded modules in the user's environment
- `module purge` - clear all modules from the user's environment


### Two Batch Script Examples

For illustration, example jobscripts for CPU and GPU applications are shown below, but each should be customised for your specific job. More examples are given in the **Applications** section.

??? note "CPU Application (with OpenMPI)"

    ```bash
    #!/bin/bash

    #SBATCH --job-name=mpi-job-name             # Specify a name for the job
    #SBATCH --output=mpi-job-output.out         # Specify where stdout will be written
    #SBATCH --error=mpi-job-error.err           # Specify where stderr will be written
    #SBATCH --time=01:30:00                     # time limit of 1hr30min
    #SBATCH --mail-user=<email address>         # Specify email address for notifications
    #SBATCH --mail-type=ALL                     # Specify types of notification (eg job Begin, End)
    #SBATCH --ntasks=32                         # Number of tasks to run (for MPI tasks this is number of cores) 
    #SBATCH --nodes=4                          # Maximum number of nodes on which to launch tasks 

    #SBATCH --partition=k2-hipri                # Specify SLURM partition to queue the job

    
    module load mpi/openmpi/4.1.1/gcc-9.3.0 # Load the application and dependencies
    module load <mpi-application>

    mpirun <application-command> -n 32 [options] # Run the code
    ```


??? note "GPU Application (MATLAB)"

    ```bash
    #!/bin/bash

    #SBATCH --job-name=matlab-job-name
    #SBATCH --output=matlab-output.out
    #SBATCH --time=00:30:00
    #SBATCH --nodes=1
    #SBATCH --ntasks=4
    #SBATCH --mem-per-cpu=5G
    #SBATCH --partition=k2-gpu
    #SBATCH --gres=gpu:1g.10gb:1

    module load matlab/R2022a

    # Ulster University (UU) users must use UU's Matlab licence by declaring
    # (removing comments) these environment variables:
    #export MLM_LICENSE_FILE=27000@193.61.190.229
    #export LM_LICENSE_FILE=27000@193.61.190.229

    matlab -nosplash -nodisplay -r "matlab_gpu_script;"
    ```


## Managing Submitted Jobs

SLURM also has commands for managing your jobs.

### Checking job status
You can view the status of your submitted jobs by using the `squeue` command:

```bash
[<username>@login1 [kelvin2] ~]$ squeue -u <username>
```
```bash
             JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
          <jobid>  k2-hipri hello-wo  <username>  R       0:02      1 node103
```

The status of your job is given by the Job State code (ST) and the nodes your job is running on (or the reason it is not running) is in the 'NODELIST(REASON)' column. Common Job State codes are:

|State Code| State | Description|
|-|-|-|
|**R**|Running |Your job is currently running.|
|**PD** |Pending|Your job is waiting for resources to be allocated|
|**F**|Failed|Your job submission has failed.|

The definitive list of job state codes are given in the [SLURM documentation](https://slurm.schedmd.com/squeue.html#SECTION_JOB-STATE-CODES){target=_blank}



### Cancelling jobs

If you need to cancel a pending or running job you can use the `scancel` command:
```bash
scancel <jobid>
```

Users can delete their own jobs only.

### Analysing previous jobs using `sacct`

Jobs you have previously ran can be analysed to check how much of the resources you had allocated were used by the program. More information may be found in the [SLURM documentation](https://slurm.schedmd.com/sacct.html){target=_blank}.

```bash
sacct -j <jobid> --format="JobID,jobname,NTasks,nodelist,CPUTime,ReqMem,MaxVMSize,Elapsed"
```


## Interactive Jobs

Interactive jobs allow the user to interact with their applications as they are running on the compute node(s). This is useful for data visualisation and for debugging programs and more complex code compilations.

!!! note

    Since interactive sessions involve waiting on human input, they are generally an inefficient use of computational resources. As such, we recommend using a batch script whenever possible to run your applications automatically in the background.

You can launch an interactive session by including the `srun` command in the terminal, followed by a list of computational resources, followed by `--pty bash`.

For example, the follow command requests 10 CPU cores and 10GB of RAM on 1 node for a 1 hour period.

```bash
[<username>@login1 [kelvin2] ~]$ srun -p k2-hipri -N 1 -n 10 --mem-per-cpu=1G --time=1:00:00 --pty bash
```
The job will then be queued and launched when resources are allocated on the system. Note that the domain on the terminal will change to a compute node.
```bash
srun: job <jobid> queued and waiting for resources
srun: job <jobid> has been allocated resources
[<username>@node103 [kelvin2] ~]$
```

Once connected to a login node you can load modules, set your environment and run your jobs interactively.
### Interactive jobs with graphical applications

It is possible to interact with graphical applications on Kelvin2 by launching a VNC server and creating an SSH tunnel.

1. Launch an interactive job on a compute node by typing the following command into your Kelvin2 terminal. Modify your resource requirements as needed.
    ```bash
    [<username>@login3 [kelvin2] ~]$ srun -p k2-hipri -N 1 -n 10 --mem-per-cpu=1G --time=1:00:00 --pty bash
    ```
    
1. On the compute node, launch a vncserver (TigerVNC) by typing the following into your Kelvin2 terminal:
    ```bash
    [<username>@node181 [kelvin2] ~]$ vncserver
    ```
    
1. If this is your first time launching a VNC server you will be prompted to set a password. For security reasons, this should be different to your login/SSH password for Kelvin2.
    ```bash
    You will require a password to access your desktops.

    Password:
    Verify:
    Would you like to enter a view-only password (y/n)? n
    A view-only password is not used
    ```
    
1. Once your vncserver is launched you will have information printed to your screen:
    ```bash
    New 'node181.pri.kelvin2.alces.network:1 (<username>)' desktop is node181.pri.kelvin2.alces.network:1

    Starting applications specified in /users/<username>/.vnc/xstartup
    Log file is /users/<username>/.vnc/node181.pri.kelvin2.alces.network:1.log
    ```

1. Take note of the number after the compute node address (e.g. the `1` at the end of `node181.pri.kelvin2.alces.network:1`). This specifies the port number you need when creating your tunnel. Since the number here is `1`, we use port `5901`, if the number `7`, we would use port `5907`.

1. In a separate terminal on your local computer, enter the following command to launch an SSH tunnel. Modify your node name and port numbers as appropriate. Here `5903` specifies the port on your local host and `5901` comes from the port number noted in the previous step. You will have to log in and authenticate using your usual credentials.


    **From inside the QUB Network**
    ```bash
    ssh -L 5903:node181.pri.kelvin2.alces.network:5901 <username>@kelvin2.qub.ac.uk
    ```

    **From outside the QUB Network**
    ```bash
    ssh -L 5903:node181.pri.kelvin2.alces.network:5901 -p 55890 -i /path/to/ssh/key <username>@login.kelvin.alces.network
    ```

1. Now open  your VNC application and connect to localhost on port 5903 (or whichever port you chose in Step 6). You will have to enter the password set on Step 3.

1. You will now have a graphical view of a terminal in Kelvin2. From here you may load modules and open graphical applications as required.

1. After you have finished your session
    - disconnect from your graphical session on your VNC application.
    - find and close the vncserver on the compute node by typing the following commands into your Kelvin2 terminal.

        ```bash
        [<username>@node181 [kelvin2] ~]$ vncserver -list

        TigerVNC server sessions:

        X DISPLAY #     PROCESS ID
        :1              102389
        [<username>@node181 [kelvin2] ~]$ vncserver -kill :1
        Killing Xvnc process ID 102389
        ```
    - Close the tunnel by exiting the local terminal opened in Step 6.

## Tips for optimising your jobscript
  - Use the job analysis tool [sacct](https://slurm.schedmd.com/sacct.html){target=_blank} to check if you allocated more memory than the necessary for futures jobs with the same application, e.g. `sacct -j <job_num> --format="JobID,jobname,NTasks,nodelist,CPUTime,ReqMem,MaxVMSize,Elapsed"`
  - Do not allocate more resources than available in the requested nodes or you job will not run.
  - When possible, try not to allocate a large number of CPUs or a large amount of memory in a single node. Instead, spread the allocation among several nodes. 20 CPUs and 100 GB of memory is a reasonable amount to be allocated in a single node.
  - Give freedom to Slurm to distribute the resources among the nodes. Use the flags `--ntasks` and `--mem-per-cpu` to allocate resources per CPU, rather than `--ntasks-per-node` or `--mem`.
  - Allocate always the correct partition, double check the resources you require, in particular the wall time, and fit it in the particular partition.
  - Specify an error output different to the standard output. In case the job crashes, this output has useful information about why the job failed, so how it can be fix.
  - Activate email notifications.
  - If you are intending to run a large number of similar jobs, consider submitting a [Job Array](https://slurm.schedmd.com/job_array.html){target=_blank}


## Slurm cheat sheets

??? note "Common job commands"

    |    Description                         | Command                                                                                        |
    |:----------------------------|:---------------------------------------------------------------------------------------------|
    | Submit a job	              | sbatch <job script>                                                                          |
    | Delete a job	              | scancel <job ID>                                                                             |
    | Job status (all)	          | squeue                                                                                       |
    | Job status (by job)	      | squeue -j <job ID>                                                                           |
    | Job status (detailed)	      | scontrol show job <job ID>                                                                   | 
    | Show expected start time   | squeue -j <job ID> --start                                                                   |
    | Start an interactive job    | srun --pty  /bin/bash                                                                        |
    | Monitor jobs resource usage | sacct -j <job_num> --format="JobID,jobname,NTasks,nodelist,CPUTime,ReqMem,MaxVMSize,Elapsed" |


??? note "Slurm Environmental variables"

    | Variable              | Function                                                                                                   |
    |:----------------------|:-----------------------------------------------------------------------------------------------------------|
    | SLURM_ARRAY_JOB_ID    | Job array's master job ID number.                                                                          |
    | SLURM_ARRAY_TASK_ID   | Job array ID (index) number.                                                                               |
    | SLURM_CLUSTER_NAME    | Name of the cluster on which the job is executing.                                                         |
    | SLURM_CPUS_PER_TASK   | Number of cpus requested per task. Only set if the --cpus-per-task option is specified.                    |
    | SLURM_JOB_ACCOUNT     | Account name associated of the job allocation.                                                             |
    | SLURM_JOB_ID          | The ID of the job allocation.                                                                              |
    | SLURM_JOB_NAME        | Name of the job.                                                                                           |
    | SLURM_JOB_NODELIST    | List of nodes allocated to the job.                                                                        |
    | SLURM_JOB_NUM_NODES   | Total number of nodes in the job's resource allocation.                                                    |
    | SLURM_JOB_PARTITION   | Name of the partition in which the job is running.                                                         |
    | SLURM_JOB_UID         | The ID of the job allocation. See SLURM_JOB_ID. Included for backwards compatibility.                      |
    | SLURM_JOB_USER        | User name of the job owner                                                                                 |
    | SLURM_MEM_PER_CPU     | Same as --mem-per-cpu                                                                                      |
    | SLURM_MEM_PER_NODE    | Same as --mem                                                                                              |
    | SLURM_NTASKS          | Same as -n, --ntasks                                                                                       |
    | SLURM_NTASKS_PER_NODE | Number of tasks requested per node. Only set if the --ntasks-per-node option is specified.                 |
    | SLURM_PROCID          | The MPI rank (or relative process ID) of the current process.                                              |

More information about  Slurm commands, flags and environment variables, can be found in the 
[Slurm web page](https://slurm.schedmd.com){target=_blank}.
