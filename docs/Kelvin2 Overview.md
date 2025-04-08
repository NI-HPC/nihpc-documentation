The NI-HPC centre hosts the Kelvin-2 research cluster.
Kelvin2 runs on a Linux (Centos 7) Operating system.
Below we will list the resources available.

## Compute

- 96 x 128 core Dell PowerEdge R6525 compute nodes with AMD EPYC 7702 dual 64-Core Processors (786GB RAM).
- 8 High memory nodes (2TB RAM).
- 32 x NVIDIA Tesla v100 GPUs in 8 nodes.
- 16 x NVIDIA Tesla A100 GPUs in 4 nodes.
- 4 x NVIDIA Tesla H100 GPUs in 1 node.
- 4 x Intel MAX 1100 GPUs in 1 node.
- 8 x AMD MI300X GPUs in 1 node.
- 2PB of lustre parallel file system for scratch storage.

## Storage

There are 4 pools of storage on Kelvin2 :

### Home
        
    /users/"username"
- Default place that users login to.
- 50GB/100K file quota.
- No automated file deletion - therefore good to store smaller/compressed longer term data.

### Scratch
        
    /mnt/scratch2/users/"username"
 - Large, shared storage area
 - No Quota
 - Temporary data solution - once per month a purge will delete any data that hasnt been accessed in the previous 3 months.
 - Mainly used for storing data that is neccassary to running jobs and storing their output.
 - Once results are output, permanent data should be moved to longer term storage ( Home/McClayRDS ).

### Temp

    /tmp

- Local scratch disk on nodes.
- Data will be automatically deleted when session ends.

### McClayRDS

    /mnt/autofs/mcclayrds-projects
    
- QUB users only
- Available only from  login and data movement nodes, not compute nodes.
- Group quotas enabled.
- Replicated to secondary site.
- To apply for storage on McClayRDS, contact the Research Data Management team [here](https://libguides.qub.ac.uk/ResearchDataManagement/Introduction).
