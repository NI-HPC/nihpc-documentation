# Compilers

Kelvin-2 has a large set of compilers and libraries for those users who compile their own self-programmed applications.
We hardly recommend avoiding the system compilers, and to use those ones which are installed as modules. Modules for compilers as flagged in general as

    compilers/<name>/<version>

And libraries

    libs/<name>/<version>/<compiler>

where <compiler> states the compiler and version that it was compiled with, and in some cases, other libraries as dependencies.
If you are going to use a precompiled library, be sure to use for your application the same compiler and version that the particular library was compiled with.

## ***Compilers available on Kelvin-2***

On Kelvin-2, for usual programming languages as C, C++, or Fortran, we recommend using the GNU compiling suite. It is well tested, and it is currently the fastest for AMD systems. 
It is also a universal compiling suite, so any code will compile with it.

    compilers/gcc/10.3.0
    compilers/gcc/11.2.0
    compilers/gcc/13.2.0
    compilers/gcc/14.1.0
    compilers/gcc/system *default*

Other compilers installed in Kelvin-2 are Clang (as part of library llvm), Nvidia nvhpc (only in Nvidia GPU nodes), AMD-Rocm (Only AMD GPU nodes).

    libs/llvm/15.0.3
    libs/llvm/17.0.1
    nvhpc/22.7
    nvhpc/22.7-byo
    nvhpc/22.7-nompi
    nvhpc/23.11
    nvhpc/24.5
    nvhpc-hpcx/23.11-cuda11
    nvhpc-hpcx/23.11-cuda12
    nvhpc-hpcx/24.5
    amd-rocm/rocm-6.2.0
    amd-rocm/rocm-6.3.3

In consistence with the general practice, jobs must not be run into the login nodes, and that includes compilation. 
The compilers check the hardware of the node where they are working, and they produce an executable adapted to the architecture of the node. 
Login nodes have different architecture than compute nodes, so an application compiled in the login nodes will likely not work in the compute nodes.

To compile, an interactive session should be opened in a compute node. 
All the compute nodes in Kelvin-2 have the same architecture, so in general it does not matter in which one of them the compilation is performed. 
High-memory nodes have more RAM memory, but the model of the RAM and the processors are the same as the standard nodes, 
so a program compiled in the standard nodes will work in the high-memory nodes.

One exception is to compile a program designed to work on the GPUs. 
In this case, the session must be allocated in the k2-gpu partition, and it must be allocated a GPU resource where the application will be executed. 
For compatibility, we recommend using the H100 GPUs to compile Nvidia CUDA codes. Further details about how to compile for GPUs will be stated in the specific section.

    srun --pty --partition=k2-gpu-interactive --time=3:00:00 --ntasks=1 --mem-per-cpu=4G --gres gpu:h100:1 bash

### ***Example of compilation***

hello_world.c

    // Program to print Hello World using C language

    #include <stdio.h>
    #include <stdlib.h>
    
    void main(void)
    {
        printf("Hello World... \n");
    }

Compile and execute

    [<user>@login1 [kelvin2] ~]$ srun --pty --partition=k2-hipri --ntasks=1 --mem-per-cpu=100M bash
    [<user>@node162 [kelvin2] ~]$ module load compilers/gcc/14.1.0
    [<user>@node162 [kelvin2] ~]$ gcc -O2 -o hello_world.x hello_world.c
    [<user>@node162 [kelvin2] ~]$ ./hello_world.x
    Hello World...

## ***Compiling parallel applications***

- OpenMP

 All the compilers have integrated the libraries to execute in parallel using the Open Multi Processor protocol.
 In the case of the GNU compiler, the OpenMP protocol is activated just adding the flag

     -fopenmp

 to the compilation command.

hello_world_omp.c

    // OpenMP program to print Hello World
    // using C language
     
    // OpenMP header
    #include <omp.h>
    #include <stdio.h>
    #include <stdlib.h>
 
    int main(int argc, char* argv[])
    {
     
        // Beginning of parallel region
        #pragma omp parallel
        {
 
            printf("Hello World... from thread = %d\n",
                   omp_get_thread_num());
        }
        // Ending of parallel region
    }

Compile and execute

    [<user>@login1 [kelvin2] ~]$ srun --pty --partition=k2-hipri --ntasks=8 --mem-per-cpu=100M bash
    [<user>@node162 [kelvin2] ~]$ module load compilers/gcc/14.1.0
    [<user>@node162 [kelvin2] ~]$ gcc -O2 -fopenmp -o hello_world_omp.x hello_world_omp.c
    [<user>@node162 [kelvin2] ~]$ ./hello_world_omp.x
    Hello World... from thread = 3
    Hello World... from thread = 2
    Hello World... from thread = 6
    Hello World... from thread = 7
    Hello World... from thread = 4
    Hello World... from thread = 5
    Hello World... from thread = 0
    Hello World... from thread = 1

- MPI

 To compile using the Message Passing Interface protocol, in Kelvin-2 it is necessary to load the modules for those libraries and change the compiling commands.
 The modules that activate the MPI libraries are flagged in Kelvin-2 as

     mpi/<name>/<version>/<compiler>

 where <compiler> points the compiler and version that the MPI libraries were compiled with.
 This is important for compatibility of the compilations, it should be used the same compiler for the application than the one that was used to compile the MPI libraries.

 The available MPI implementations on Kelvin-2 are

    mpi/openmpi/4.1.6/gcc-8.5.0
    mpi/openmpi/5.0.3/gcc-13.2.0+nvidia-cuda-12.8.0
    mpi/openmpi/5.0.3/gcc-14.1.0 *default*
    mpi/openmpi/5.0.3/gcc-8.5.0
    mpi/openmpi/5.0.7/gcc-14.1.0+rocm
    mpi/intel-mpi/2016u1/bin
    mpi/intel-mpi/2020u4/bin
    mpi/mvapich2/2.3.4/gdr
    mpi/mvapich2/2.3.5/gdr
    mpi/mvapich2/2.3.5/gdr+slurm

 We hardly recommend using the OpenMPI compiling suite, it has been widely tested, and it works stable on Kelvin-2.
 The MPICh suite has not been so deeply tested, and it is not guaranteed that applications compiled with it will work on Kelvin-2.
 If you decide to use the MPICh suite, be sure to test your application before using it for production.

 To compile using MPI, the compiling commands should be changed.
 For example, to compile a C program with the compiler GNU 9.3.0, the command to be used is
 
     gcc <flags> -o my_executable.x my_C_program.c
 
 And if we want to compile it in parallel using the MPI version 4.1.1, the compiling command should be changed to

     mpicc <flags> -o my_executable.x my_MPI-C_program.c

 In general, the compiling commands should be changed

     gcc -> mpicc
     g++ -> mpicxx
     gfortran -> mpifortran

 These values are taken by default, and the command "mpicc" will use the GNU compiler and the MPI libraries.
 If you want to use a different compiler, you have to specify it in the environment variables

    OMPI_MPICC
    OMPI_MPICXX
    OMPI_FC

 For example, to compile using the MPI library v5.0.3 with the clang compiler included in the module llvm v17.0.1, the environment variables should be defined as

    export OMPI_MPICC=clang
    export OMPI_MPICXX=clang++
    export OMPI_FC=flang

 and compile using the commands "mpicc", "mpicxx", "mpifortran".

 hello_world_mpi.c

    #include <mpi.h>
    #include <stdio.h>

    int main(int argc, char** argv) {
        // Initialize the MPI environment
        MPI_Init(NULL, NULL);

        // Get the number of processes
        int world_size;
        MPI_Comm_size(MPI_COMM_WORLD, &world_size);

        // Get the rank of the process
        int world_rank;
        MPI_Comm_rank(MPI_COMM_WORLD, &world_rank);

        // Get the name of the processor
        char processor_name[MPI_MAX_PROCESSOR_NAME];
        int name_len;
        MPI_Get_processor_name(processor_name, &name_len);

        // Print off a hello world message
        printf("Hello world from processor %s, rank %d out of %d processors\n",
               processor_name, world_rank, world_size);

        // Finalize the MPI environment.
        MPI_Finalize();
    }

Compile and execute

    [<user>@login1 [kelvin2] ~]$ srun --pty --partition=k2-hipri --ntasks=8 --mem-per-cpu=100M bash
    [<user>@node162 [kelvin2] ~]$ module load compilers/gcc/14.1.0
    [<user>@node162 [kelvin2] ~]$ module load mpi/openmpi/5.0.3/gcc-14.1.0
    [<user>@node162 [kelvin2] ~]$ mpicc -O2 -o hello_world_mpi.x hello_world_mpi.c
    [<user>@node162 [kelvin2] ~]$ mpirun -np 8 ./hello_world_mpi.x
    Hello world from processor node162.pri.kelvin2.alces.network, rank 0 out of 8 processors
    Hello world from processor node162.pri.kelvin2.alces.network, rank 4 out of 8 processors
    Hello world from processor node162.pri.kelvin2.alces.network, rank 5 out of 8 processors
    Hello world from processor node162.pri.kelvin2.alces.network, rank 1 out of 8 processors
    Hello world from processor node162.pri.kelvin2.alces.network, rank 3 out of 8 processors
    Hello world from processor node162.pri.kelvin2.alces.network, rank 6 out of 8 processors
    Hello world from processor node162.pri.kelvin2.alces.network, rank 7 out of 8 processors
    Hello world from processor node162.pri.kelvin2.alces.network, rank 2 out of 8 processors

- OpenACC

 Open accelerator is a feature included in the Nvidia compiler.
 It can be activated adding to the compilation sequence the flag

     -acc

 This option implies an intelligent check-up of the hardware at the time of the execution.
 The compiler will use the available hardware resources at that time to optimize the acceleration of the marked regions of the code as "OpenACC",
 multiple-CPUs or GPUs will be allocated by the compiler for the execution, the user does not need to specify the resources at the time of the execution.

 OpenACC is a feature of the Nvidia compiler. On Kelvin-2, it will work only on the GPU nodes.
 If you want the compiler to take advantage of the GPU accelerators, you must allocate a GPU resource when you compile the code, preferably an A100 GPU.

 More information about OpenACC, including pdf user guides can be found in the web site </br>
 [https://www.openacc.org](https://www.openacc.org){target=_blank}

 hello_world_openacc.c

    #include <stdio.h>
    #ifdef _OPENACC
    #include <openacc.h>
    #endif

    int main(void) {
    #ifdef _OPENACC
        acc_device_t devtype;
    #endif

        printf("Hello world from OpenACC\n");
    #ifdef _OPENACC
        devtype = acc_get_device_type();
        printf("Number of available OpenACC devices: %d\n", acc_get_num_devices(devtype));
        printf("Type of available OpenACC devices: %d\n", devtype);
    #else
        printf("Code compiled without OpenACC\n");
    #endif

        return 0;
    }

Compile and execute

    [<user>@login1 [kelvin2] ~]$ srun --pty --partition=k2-gpu-interactive --time=3:00:00 --ntasks=1 --mem-per-cpu=1G --gres gpu:h100:1 bash
    [<user>@gpu111 [kelvin2] ~]$ module load nvhpc/24.5
    [<user>@gpu111 [kelvin2] ~]$ nvc -O2 -acc -o hello_world_openacc.x hello_world_openacc.c
    [<user>@gpu111 [kelvin2] ~]$ ./hello_world_openacc.x
    Hello world from OpenACC
    Number of available OpenACC devices: 1
    Type of available OpenACC devices: 4

## ***Compiling applications that use GPUs***

To compile a program designed to work on the Graphical Processing Units of Kelvin-2, it is essential that it is compiled in a GPU node, so the queue "k2-gpu" must be allocated.
For backwards compatibility, the program must be compiled in the latest model of GPU present in the machine, in our case, the Nvidia H100 GPUs.
So, when allocating the interactive session to carry out the compilation, the resource A100 should be allocated with the flag

    --gres gpu:h100:1

During the last years, the popularity of the specific GPU-focused programming languages, such as CUDA, has decreased.
This is due to the publicly-available libraries have become more and more complete, and practically any mathematical operation that can benefit of the acceleration advantages of a GPU is present on those libraries.
Some examples are "cublas" for linear-algebra operations, and "cufft" for Fourier transforms.
Most of the libraries designed to work in the GPUs of Kelvin-2 can be found in the module for the Nvidia CUDA drivers:

    libs/nvidia-cuda/11.0.3/bin
    libs/nvidia-cuda/11.1.1/bin
    libs/nvidia-cuda/11.6.2/bin
    libs/nvidia-cuda/11.8.0/bin
    libs/nvidia-cuda/12.4.0/bin
    libs/nvidia-cuda/12.8.0/bin
    libs/nvidia-cuda/9.0.176.4/bin

These libraries can be including in the executables, as usual adding the flag "-l" to the compilation command, for example

    gcc <flags> -lcublas -lcufft -o my_executable.x my_GPU_program.c

Nevertheless, if you require a very specific operation, that is not present in the available libraries, and it is necessary to use CUDA, the Nvidia compiler can be used.
Currently, the Nvidia compiler is the only one installed on Kelvin-2 that can compile CUDA code

    nvhpc/22.7
    nvhpc/22.7-byo
    nvhpc/22.7-nompi
    nvhpc/23.11
    nvhpc/24.5
    nvhpc-hpcx/23.11-cuda11
    nvhpc-hpcx/23.11-cuda12
    nvhpc-hpcx/24.5

This compiler can recognise the sections of the code in an intelligent way, so there is no need of more flags to point that it includes CUDA routines.
To compile with Nvidia compiler, just use its usual commands

    nvc <flags> -o my_executable.x my_CUDA_C_program.c
    nvc++ <flags> -o my_executable.x my_CUDA_C++_program.cxx
    nvfortran <flags> -o my_executable.x my_CUDA_Fortran_program.for

