# Kelvin2 Hardware

## CPU Nodes

**AMD Nodes**

- **Nodes:** node[101-209]
- **Server:** Dell PowerEdge R6525
- **CPU:** 2x AMD EPYC 7702 64-Core Processor
- **Sockets:Cores:Threads:** 2:64:1
- **Memory:**
    - node[101-160]: 768GiB
    - node[161-182], node[186-209]: 1TiB
    - node[183-185]: 960GiB
- **NUMA domains:**
    - node[101-133,135-160]: 8
    - node[134,161-209]: 2
- **SLURM partitions:**
    - node[101-159], node[161-165], node[167-182], node[196-209]: k2-hipri, k2-medpri, k2-lowpri, k2-epsrc
    - node160: k2-sandbox
    - node166, node[183-195]: (reserved for specific research groups)

**AMD X Series Nodes**

- **Nodes:** node[301-302]
- **Server:** Dell PowerEdge R6525
- **CPU:** 2x AMD EPYC 7773X 64-Core Processor
- **Sockets:Cores:Threads:** 2:64:1
- **Memory:** 1TiB
- **NUMA domains:** 8
- **SLURM partitions:** k2-amd-xseries

## GPU Nodes

**V100 nodes**

- **Nodes:** gpu[103-110]
- **Server:** Dell EMC DSS 8440
- **CPU:** 2x Intel Xeon Platinum 8168 CPU @ 2.70GHz
- **Sockets:Cores:Threads:** 2:24:1
- **Memory:** 512GiB
- **NUMA domains:** 2
- **GPU:** 4x Tesla V100 PCIe 32GB
- **SLURM partitions:** k2-gpu-v100, k2-gpu-interactive, k2-epsrc-gpu-v100

**A100 nodes**

- **Nodes:** gpu[111-113]
- **Server:** Dell PowerEdge XE8545
- **CPU:** 2x AMD EPYC 7763 64-Core Processor
- **Sockets:Cores:Threads:** 2:64:1
- **Memory:** 1TiB
- **NUMA domains:** 8
- **GPU:** 4x Tesla A100 SXM4 80GB
- **SLURM partitions:** k2-gpu-a100, k2-gpu-interactive, k2-epsrc-gpu-a100

**A100 slice node**

- **Nodes:** gpu[114]
- **Server:** Dell PowerEdge XE8545
- **CPU:** 2x AMD EPYC 7763 64-Core Processor
- **Sockets:Cores:Threads:** 2:64:1
- **Memory:** 1TiB
- **NUMA domains:** 8
- **GPU:** 4x Tesla A100 SXM4 80GB, each partitioned into 7 instances
- **SLURM partitions:** k2-gpu-a100mig, k2-gpu-interactive, k2-epsrc-gpu-h100

**H100 node**

- **Nodes:** gpu[121]
- **Server:** Dell PowerEdge XE8640
- **CPU:** 2x Intel(R) Xeon(R) Platinum 8468 Processor
- **Sockets:Cores:Threads:** 2:96:1
- **Memory:** 1TiB
- **NUMA domains:** 8
- **GPU:** 4x Tesla H100 SXM5 80GB
- **SLURM partitions:** k2-gpu-h100, k2-gpu-interactive, k2-epsrc-gpu-h100

**MI300X node**

- **Nodes:** gpu[123]
- **Server:** Dell PowerEdge XE9680
- **CPU:** 2x Intel(R) Xeon(R) Platinum 8470 Processor
- **Sockets:Cores:Threads:** 2:104:1
- **Memory:** 2TiB
- **NUMA domains:** 8
- **GPU:** 8 x AMD MI300x
- **SLURM partitions:** k2-gpu-amd, k2-gpu-interactive, k2-epsrc-gpu-amd

**MAX 1100 node**

- **Nodes:** gpu[122]
- **Server:** Dell PowerEdge R760xa
- **CPU:** 2x Intel(R) Xeon(R) Gold 6438Y+ Processor
- **Sockets:Cores:Threads:** 2:64:1
- **Memory:** 500Gb
- **NUMA domains:** 8
- **GPU:** 4x Ponte Vecchio XT (1 Tile) Data Center GPU Max 1100
- **SLURM partitions:** k2-gpu-intel, k2-gpu-interactive, k2-epsrc-gpu-intel


## High Memory Nodes

- **Nodes:** smp[106-113]
- **Server:** Dell PowerEdge R6525
- **CPU:** 2x AMD EPYC 7702 64-Core Processor
- **Sockets:Cores:Threads:**
    - smp[106-107]: 8:16:2
    - smp[108-113]: 2:64:1
- **Memory:** 2TiB
- **NUMA domains:** 
    - smp[106-109]: 8
    - smp[110-113]: 2
- **SLURM partitions:** k2-himem, k2-epsrc-himem

## Legacy CPU Nodes

**Type 1**

- **Nodes:** node[001-024], node[026-034], node[036-039], node[041], node[045-051], node[053-061]
- **Server:** ProLiant XL170r Gen9
- **CPU:** Intel Xeon CPU E5-2660 v3 @ 2.60GHz
- **Sockets:Cores:Threads:** 2:10:1
- **Memory:**
    - node[001-014]: 256GiB
    - All except node[001-014]: 128GiB
- **NUMA domains:** 2
- **SLURM partitions:** 
    - all except node[061]: hipri, medpri, lowpri
    - node[061]: sandbox

**Type 2**

- **Nodes:** node[071-079]
- **Server:** ProLiant XL170r Gen10
- **CPU:** Intel Xeon Gold 6126 CPU @ 2.60GHz
- **Sockets:Cores:Threads:** 2:12:1
- **Memory:**
    - node[071-073]: 384GiB
    - node[074-079]: 192GiB
- **NUMA domains:** 2
- **SLURM partitions:** hipri, medpri, lowpri, avx512

## Legacy GPU Nodes

**K4200 node**

- **Nodes:** gpu01
- **Server:** ProLiant DL380 Gen9
- **CPU:** 2x Intel Xeon CPU E5-2640 v3 @ 2.60GHz
- **Sockets:Cores:Threads:** 2:8:1
- **Memory:** 32GiB
- **NUMA domains:** 2
- **GPU:** Quadro K4200
- **SLURM partitions:** gpu

**V100 node**

- **Nodes:** gpu02
- **Server:** ProLiant XL270d Gen10
- **CPU:** 2x Intel Xeon Gold 6126 CPU @ 2.60GHz
- **Sockets:Cores:Threads:** 2:12:1
- **Memory:** 768GiB
- **NUMA domains:** 2
- **GPU:** 4x Tesla V100 SXM2 32GB
- **SLURM partitions:** gpu

## Legacy High Memory Nodes

**Type 1**

- **Nodes:** smp01
- **Server:** ProLiant DL560 Gen9
- **CPU:** 4x Intel Xeon CPU E5-4627 v3 @ 2.60GHz
- **Sockets:Cores:Threads:** 4:10:1
- **Memory:** 1TiB
- **NUMA domains:** 4
- **SLURM partitions:** himem

**Type 2**

- **Nodes:** smp[03-04]
- **Server:** ProLiant DL360 Gen10
- **CPU:** 2x Intel Xeon Gold 6126 CPU @ 2.60GHz
- **Sockets:Cores:Threads:** 2:12:2
- **Memory:** 1TiB
- **NUMA domains:** 2
- **SLURM partitions:** himem

**Type 3**

- **Nodes:** smp05
- **Server:** ProLiant DL360 Gen10
- **CPU:** 2x Intel Xeon Gold 6130 CPU @ 2.10GHz
- **Sockets:Cores:Threads:** 2:16:1
- **Memory:** 768TiB
- **NUMA domains:** 2
- **SLURM partitions:** himem
