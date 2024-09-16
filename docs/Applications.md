# Applications

Here we will provide information on some of the most popular centrally installed applications and software tools on Kelvin2.

## **Matlab**

MATLAB is a proprietary multi-paradigm programming language and numeric computing environment developed by MathWorks. MATLAB allows matrix manipulations, plotting of functions and data, implementation of algorithms, creation of user interfaces, and interfacing with programs written in other languages.

How to load the application:

``` bash
module load matlab/R2022a
```

If X11 forwarding is enabled, running the command `matlab` will open the Matlab's GUI (it works in Mobaxterm). Otherwise, launch Matlab in console mode as follows:

``` bash
matlab -nosplash -nodisplay
```

However, please, **==DO NOT RUN MATLAB NOR ANY OTHER SOFTWARE ON LOGIN NODES==**. 

### ***Running Matlab interactively: srun + parfor (CPUs)***

One of the two correct ways to run Matlab is to request a compute node (interactively) with the `srun` command.

For example, request 1 compute node and 10 cores in "k2-hipri" partition:

``` bash
srun -p k2-hipri -N 1 -n 10 --mem=10G --time=1:00:00 --pty bash
module load matlab/R2022a
matlab -nosplash -nodisplay
```

Then, inside Matlab, notice that calling the function `feature` (line #1 below) must show that exactly 10 CPU cores are available, corresponding to the number of cores allocated above with the `srun` command. Next, the codes launches the parallel pool (`parpool`) for the "local" cluster requesting the same amount of workers as the cores allocated in this example.

``` matlab linenums="1"
feature('numcores')
p = parpool('local', 10) % 10 cores requested in this example
```

More robust, the number of workers can be read and set automatically to launch the parallel pool:

``` matlab
num_workers = str2double(getenv('SLURM_CPUS_ON_NODE'))
p = parpool('local', num_workers)
```

Finally, you can run your Matlab parallel code, which must include a "parfor" loop. For example, the following code illustrates the use of `parfor` and Monte Carlo simulation to calculate an approximate value for \( \pi \), using the formula (line #7):

$$
\pi \approx \lim_{N_{MC} \to \infty} {4 \sum_{n=1}^{N_{MC}} I(x_i^2 + y_i^2 < 1.0) \over N_{MC}}; x_i, y_i \sim \mathcal{U}(0,1),
$$

where the symbol \( \mathcal{U}(0,1) \) represents the random uniform distribution for the indicated interval and \( I(boolean) \) is an indicator function.

``` matlab title="Matlab code for parfor demonstration" linenums="1"
N = 1e6;
out = zeros(1, num_workers);
parfor i = 1:num_workers
	xy = rand(N,2);
	out(i) = sum(sum(xy.^2,2)<=1);
end
mypi = 4*sum(out)/(N*num_workers)
```

At the end of the parallel computations, the allocated parpool within Matlab can be released with the following command:

``` matlab
delete(gcp('nocreate'))
```

### ***Running Matlab with GUI directly on a compute node***

First, get a compute node and launch vnc server from the node, for example:

``` bash
srun -p k2-hipri -N 1 -n 6 --mem=10G --time=1:00:00 --pty bash
vncserver
```

In this case, let us assume that the output of `vncserver` is

*New 'node117.pri.kelvin2.alces.network:1 (jsan)' desktop is node117.pri.kelvin2.alces.network:1*

*Starting applications specified in /users/jsan/.vnc/xstartup*<br>
*Log file is /users/jsan/.vnc/node117.pri.kelvin2.alces.network:1.log*

Then, open a local terminal and launch a forward tunnel to the compute node by following these steps:

1. Go to the directory which contains the kelvin key in your PC/laptop
``` bash	
cd /drives/c/Users/jsan/.ssh
```
2. Create the tunnel (in this example illustrated with the command below, all input sent via port *5903* on your local host is being forwarded via port *5901* to the compute node *"node117.pri.kelvin2.alces.network"*. If the `vncserver` output above were *"node117.pri.kelvin2.alces.network:7"*, then the port number will be *5907* instead of *5901*. Clearly, users must replace the username *"jsan"* and the key's filename by the corresponding information for their accounts)
``` bash	
ssh -L 5903:node117.pri.kelvin2.alces.network:5901 -p 55890 -i ./kelvin-key jsan@login.kelvin.alces.network
```
3. Connect to the tunnel using your installed VNC application (the example shown in the figure below uses TurboVNC in Windows OS. More details or troubleshooting can be found in the VNC session)
![Image title](assets/TurboVNC.jpg)
4. Once in the opened terminal for the connected compute node, launch the matlab application:
```bash
module load matlab/R2022a
matlab
```

This time matlab GUI will be opened as shown in the figure below. For a better experience, use the "full screen" button in the VNC toolbar, and use the combination keys ++ctrl+alt+shift++ + F to escape from the full screen mode. Note also that using the instruction `feature('numcores')` inside the Matlab's GUI session shows correctly the number of allocated CPU cores in the compute node (CPU cores equal to 6 in this example).

![Image title](assets/TurboVNC_Matlab.jpg){ width="800" }

### ***Running Matlab with the use of GPUs***

First, get a GPU node and launch vnc server from the node, for example:

``` bash
srun -p k2-gpu -N 1 -n 4 --gres gpu:1g.10gb:1 --time=3:00:00 --mem=20G --pty bash
vncserver
```

Then, similarly as done in the previous section, (1) open a local terminal and launch a forward tunnel to the GPU node; (2) use a VNC application to connect to the tunnel; (3) load and launch the Matlab software.

To test the use of the GPU device within Matlab code, copy and paste the script file `matlab_gpu_example.m` below.

``` matlab title="matlab_gpu_example.m" linenums="1"
% The Mandelbrot algorithm iterates over a grid of real and imaginary parts.
% The following code defines the number of iterations, grid size, and grid limits.
maxIterations = 500;
gridSize = 1000;
xlim = [-0.748766713922161, -0.748766707771757];
ylim = [ 0.123640844894862,  0.123640851045266];

% Use the gpuArray function to transfer data to the GPU and create a gpuArray object.
x = gpuArray.linspace(xlim(1), xlim(2), gridSize);
y = gpuArray.linspace(ylim(1), ylim(2), gridSize);

% Many Matlab functions support gpuArrays and run directly on the GPU (e.g., meshgrid)
% The function "ones", below, create an array of ones directly on the GPU
[xGrid,yGrid] = meshgrid(x,y);
z0 = xGrid + 1i*yGrid;
count = ones(size(z0), 'gpuArray');

% The code below implements the Mandelbrot algorithm, fully running on the GPU
z = z0;
for n = 0:maxIterations
    z = z.*z + z0;
    inside = abs(z) <= 2;
    count = count + inside;
end
count = log(count);

% Plot the results
imagesc(x, y, count);
colormap([jet(); flipud(jet()); 0 0 0]);
axis off;
```

The following results should appear when you run this script in the GUI that is running on the cluster's GPU, as per below. As shown in the Matlab command window, the output of calling the function `whos` reveal the many variables (gpuArray objects) that are still allocated on the GPU.

![Image title](assets/cluster_gpu_matlab.png){ width="1000" }

### ***Running Matlab in background: sbatch***

Finally, the most convenient way to use the cluster resources may be to perform Matlab calculations in background, using `sbatch` functionality instead of `srun`. This is critical, mainly for analysis where calculations may take several hours or days. To demonstrate this, simply copy and paste the following "sbatch" script, called `gpu_example.sh`, as an example which relies on the same script prepared above to calculate the Mandelbrot solution.

``` bash title="gpu_example.sh" linenums="1"
#!/bin/bash

#SBATCH --job-name=test
#SBATCH -N 1
#SBATCH -n 4
#SBATCH --mem=20G
#SBATCH --time=00:10:00
#SBATCH --partition=k2-gpu
#SBATCH --gres=gpu:1g.10gb:1
#SBATCH --output=test_%j.log

module load matlab/R2022a

# Ulster University (UU) users must use UU's licence by declaring
# (removing comments) these environmet variables:
#export MLM_LICENSE_FILE=27000@193.61.190.229
#export LM_LICENSE_FILE=27000@193.61.190.229

matlab -nosplash -nodisplay -r "matlab_gpu_example; saveas(gcf, 'Mandelbrot'); exit;"
```
In the code above, line #19, Matlab is called in "nosplash" and "nodisplay" mode to execute the `matlab_gpu_example.m` script. Besides, note that the code will run in background, i.e., without display, therefore it must use some Matlab function like `saveas`, as shown in the code, which saves the graphical output. As a result, the Matlab file that must be listed now in the working directory, `Mandelbrot.fig`, should contain the visual results. Finally, to run this "sbatch" script, run the following command in your terminal.

``` bash
sbatch gpu_example.sh
```

## **Anaconda**

Anaconda is a software that allows the users to manage environments to install local libraries and software, particularly Python and R programming languages, for scientific computing. Thus, greatly easing software management and reducing possible incompatibilities among different tools, libraries or software versions. Kelvin2 offers to users the following modules:

``` bash title="Anaconda modules"
apps/anaconda/2.5.0/bin
apps/anaconda3/2021.05/bin
apps/anaconda3/2022.10/bin
apps/anaconda3/5.2.0/bin
```

## **Python**
Python is a high-level, multi-purpose programming language that support several paradigms such as structural, functional and object-oriented programming, together with garbage collection, in order to support code readability and fast development. With the provision of multiple especialized libraries, Python has become one of the most preferred languages for scientific programming, including machine learning and artificial intelligence, as well as bioinformatics among many other applications.

Kelvin2 supports the following modules for Python users:
``` bash title="Python modules"
apps/python/2.7.8/gcc-4.8.5
apps/python3/3.10.0/gcc-4.8.5
apps/python3/3.10.5/gcc-9.3.0
apps/python3/3.4.3/gcc-4.8.5
apps/python3/3.5.2/gcc-4.8.5
apps/python3/3.6.4/gcc-4.8.5
apps/python3/3.7.4/gcc-4.8.5
apps/python3/3.7.9/gcc-10.2.0
apps/python3/3.8.5/gcc-4.8.5
```

Notice that above are listed different Python's versions which in turn rely on different versions of the GCC compiler. The selected module must correspond to the tools or analysis pipeline that programmers have in mind, as some analysis and Python libraries may require older or newer versions of the Python/GCC software.

The next code snippet illustrates simply how to load a module and install a package in the default location (gridware folder located in the users' home folder):
``` bash
module load apps/python3/3.10.5/gcc-9.3.0
python3 -m pip install reportseff
reportseff -u $USER
```

Note: On internet blogs/forums, users will often find the recommendation ```pip install <package name>``` to install a particular tool. In this case, for the same install in Kelvin2, it is recommended to precede the command with "python3 -m" because by default pip will refer to the Python 2.7 version, which is always available from command line in Kelvin2. That is, always use ```python3 -m pip install <package name>``` to install the package.

In the previous example, reportseff is a very usefull tool to monitor the efficient utilization of cluster resources (time, RAM, CPU/GPU) that may be critical for your work. It should be used in addition to ```sacct``` command, because it helps to optimize your jobs which may translates on significantly lower queue times. For instance, the reportseff's outcome below shows some job statistics (greed/red colour codes highlight good/poor resources utilization) for selected jobs.

![Image title](assets/reportseff.png){ width="500" }

This approach, using directly Python modules and installing with pip (Python's PyPI), is mainly recommended for simpler or only-one-time analysis, or tests. Particularly, it has two main drawbacks which are discussed next together with recommendations:

1. It uses the home folder quota (~50 GB hard disk + 100,000 files limit)<br>
To sort this out, users must declare/modify the environment variables "PATH" and "PYTHONPATH". For example,
``` bash
mkdir /mnt/scratch2/users/$USER/gridware
export PATH=/mnt/scratch2/users/$USER/gridware/bin/:$PATH
export PYTHONPATH=/mnt/scratch2/users/$USER/gridware/site-packages/:$PYTHONPATH
```
creates and uses the folder "gridware" in users' scratch folder, so the pip install will be redirected to the scratch instead of home folder.<br>
2. Some Python's tools will need also to install libraries in the default locations, e.g., /user/local<br>
The issue is that users do not have write permission in Kelvin2 system paths. In this case, as well as in the case that users need to install their own version of Python or other supporting libraries, it is strongly recommended to use Anaconda environments as discussed in the next section.

## **Python with Anaconda**

Similar as with the use of Python modules above, it is recommended that users redirect the default installation paths to their scratch folder to save the quota limited resources. For instance, this can be done by modifying the environment variables "CONDA_PKGS_DIRS" and "CONDA_ENVS_PATH":

``` bash
mkdir /mnt/scratch2/users/$USER/conda
export CONDA_PKGS_DIRS=/mnt/scratch2/users/$USER/conda/pkgs
export CONDA_ENVS_PATH=/mnt/scratch2/users/$USER/conda/envs
```

In the examples below, it is very critical to select the hardware correctly (CPU or GPU) before doing the installation. Therefore, before installing the needed tools, use ```srun``` as shown below to select the compute node with the convenient hardware, i.e., correspondingly in any of the CPU/GPU  partitions available in Kelvin2.

For example, failing to select a compute node with GPU device to install the Python's package may cause that installed tools do not recognize the GPU device on runtime; therefore, your program may crash or, worse, run in the CPU cores by default giving a false sense of security. The second case is very critical because your jobs may last significantly longer to perform the same computations, but also they would waste GPU resources by running only on the CPU cores. At the same time, other users will be unable to use the allocated GPU device(s).

### ***Example 1: Installing Pytorch and Ray tune***

In this example, Pytorch is installed together with Raytune. The latter is a tool used for deep learning models optimization as it helps with the evaluation and selection of model hyperparameters (e.g., number of layers, neurons per layer, selection between different transfer or optimization functions, etc.)

``` bash title="Pytorch-Raytune install" linenums="1"
srun -p k2-gpu -N 1 -n 1 --gres gpu:1g.10gb:1 --time=3:00:00 --mem=20G --pty bash
module load apps/anaconda3/2021.05/bin
module load libs/nvidia-cuda/11.7.0/bin
conda create --name py39torchRayA100 python=3.9
source activate py39torchRayA100
(py39torchRayA100) conda install pytorch torchvision torchaudio pytorch-cuda=11.7 -c pytorch -c nvidia
(py39torchRayA100) python3
(py39torchRayA100) conda install pytorch-lightning -c conda-forge
(py39torchRayA100) conda install -c conda-forge "ray-air"
(py39torchRayA100) conda deactivate
```

Line 1 above shows the use of GPU slice partitions: ```--gres gpu:1g.10gb:1```. As currently there are available 28 slices in Kelvin2, this partition is much less busy than the other GPU partitions, and at the same time would allow the users to install software in the A100 GPU devices, which would guarantee backward compatibility with GPU devices.

Lines 2-5 setup the environment variables, create an Anaconda environment named ```py39torchRayA100``` while installing Python version 3.9, and activate the environment. This Python version is strictly required here as current Raytune installation version may crash with newer Python versions than 3.10.

The next lines 6-10 are needed to install the required tools, particularly Raytune is installed in line 9. Here, the package "ray-air" contains most of the provided functionality by Raytune (Data, Train, Tune, Serve, etc.).

After installation, users must check that Pytorch can correctly utilize the GPU resources available. For example, users must see something similar to the following output when running the Pytorch's functions:

``` bash title="Testing that torch can see GPU resources"
source activate py39torchRayA100
(py39torchRayA100) python3
>>> import torch
>>> torch.cuda.is_available()
True
>>> torch.cuda.current_device()
0
>>> torch.cuda.device(0)
<torch.cuda.device object at 0x7fec6d471850>
>>> torch.cuda.get_device_name(0)
'NVIDIA A100-SXM4-80GB MIG 1g.10gb'
>>> exit()
(py39torchRayA100) conda deactivate
```

### ***Example 2: Installing tensorflow-gpu and Raytune***

Similarly, for installing tensorflow-gpu and Raytune, follow these instructions:

``` bash title="Tensorflow-Raytune install" linenums="1"
srun -p k2-gpu -N 1 -n 1 --gres gpu:1g.10gb:1 --time=3:00:00 --mem=20G --pty bash
module load apps/anaconda3/2021.05/bin
module load libs/nvidia-cuda/11.7.0/bin
conda create --name tensorflowRayA100 python=3.9
source activate tensorflowRayA100
(tensorflowRayA100) conda install -c anaconda tensorflow-gpu
(tensorflowRayA100) conda install -c conda-forge "ray-air"
(tensorflowRayA100) conda deactivate
```

As above, users also must test that tensorflow can correctly see the GPU resources. For that purpose, after installing the software, use the following commands:

``` bash title="Testing that tensorflow can see GPU resources"
source activate tensorflowRayA100
(tensorflowRayA100) python3 -c "import tensorflow as tf; print(tf.config.list_physical_devices('GPU'))"
(tensorflowRayA100) conda deactivate
```

### ***Example 3: Installing Bindsnet***

In this example, users can overview how to install Bindsnet in Kelvin2:

``` bash title="Bindsnet install" linenums="1"
srun -p k2-gpu -N 1 -n 1 --gres gpu:1g.10gb:1 --time=3:00:00 --mem=20G --pty bash
module load apps/anaconda3/2021.05/bin
conda create --name bindsnet
export PATH=/mnt/scratch2/users/$USER/conda/envs/bindsnet/bin/:$PATH
source activate bindsnet
(bindsnet) conda install python=3.10
(bindsnet) python3 -m pip install git+https://github.com/BindsNET/bindsnet.git
(bindsnet) python3 -m pip show bindsnet
(bindsnet) python3 -m pip install pytest
(bindsnet) conda deactivate
```

The apparent simplicity of above instructions hides some complexities. For example, the declaration of the "PATH" environment variable in line 4 may be neccesary when PyPi/pip is combined with Anaconda, as some of the installed packages may still be missing after installation, unexpectedly. Also, in line 6, Python's version 3.10 was installed as Bidsnet most recent version at this moment seems not to be compatible with older Python versions.

Finally, when installing Bindsnet in line 7, it automatically finds and installs all needed dependencies, including the compatible versions for torch, torchvision, etc. Therefore, we do not recommend to install any package before bindsnet, otherwise bindsnet installation can fail. Then, in line 8, the command ```python3 -m pip show bindsnet``` allows to get the physical path where Bindsnet is installed, which may be necessary to access some folders with testing examples and scripts. For the same reasons, users may want to install the package "pytest", as it is done in line 9.

As Bindsnet will install automatically Pytorch as one of its dependencies, we strongly recommend to test that Pytorch can see the GPU resources, as discussed in Example 1.

The following is an example of sbatch script that uses the installed Bindsnet package, which can be launched from command line using the command ```sbatch bindsnet_example.sh```. Of course, users must have first to prepare their python codes, with the main file name provided in line 19. 

``` bash title="bindsnet_example.sh" linenums="1"
#!/bin/bash
#SBATCH --job-name=nnet
#SBATCH -N 1
#SBATCH -n 4
#SBATCH --mem=40G
#SBATCH --partition=k2-gpu
#SBATCH --gres gpu:v100:1
##SBATCH --gres gpu:a100:1
#SBATCH --time=03:00:00
#SBATCH --output=nnet_%j.log

module load apps/anaconda3/2021.05/bin
export CONDA_PKGS_DIRS=/mnt/scratch2/users/$USER/conda/pkgs
export CONDA_ENVS_PATH=/mnt/scratch2/users/$USER/conda/envs
export PATH=/mnt/scratch2/users/$USER/conda/envs/bindsnet/bin/:$PATH

source activate /mnt/scratch2/users/$USER/conda/envs/bindsnet

python3 <code_file_name>.py
```

<br>**==VERY IMPORTANT==**: The three examples above were discussed to highlight the complexities of installing packages in Python and Anaconda, mainly when there may be issues related to package version incompatibilities, the proper Python's version (maybe an older or newer version is strictly required), package installation order, or the possible necessity to install supporting libraries.

## **Jupyter notebook**

Here, we present simple instructions to install and use Jupyter notebook in Kelvin2, in the case for training deep learning networks with tensorflow-gpu.

``` bash title="Jupyter notebook installation" linenums="1"
srun -p k2-gpu -N 1 -n 4 --gres gpu:1g.10gb:1 --time=3:00:00 --mem=20G --pty bash
#srun -p k2-hipri -N 1 -n 4 --time=3:00:00 --mem=16G --pty bash
module load apps/anaconda3/2021.05/bin
conda create -n tf-gpu tensorflow-gpu
# export PATH=/mnt/scratch2/users/$USER/conda/envs/tf-gpu/bin/:$PATH
source activate tf-gpu
(tf-gpu) conda install -c anaconda jupyter
(tf-gpu) conda deactivate
```

If the user does not need to use GPU resources, for example, the plan is to run on CPU cores, then ```srun``` instruction in line 2 must be used, instead of line 1, to get an interactive session in CPU nodes for the installation. Also, ignore the installation of tensorflow part if the only purpose is to install Jupyter notebook to be used with other packages, such as Pandas or numpy. In that case, just create the environment in line 4 with the following instruction: ```conda create -n <environment_name>```.

Finally, to access Jupyter notebook remotely from the (local) browser in the user's PC/laptop, the user must launch a Jupyter server connection and create a tunnel to it. To lauch the server, run these commands:

``` bash linenums="1"
srun -p k2-gpu -N 1 -n 4 --gres gpu:1g.10gb:1 --time=3:00:00 --mem=20G --pty bash
#srun -p k2-hipri -N 1 -n 4 --time=3:00:00 --mem=16G --pty bash
module load apps/anaconda3/2021.05/bin
source activate tf-gpu
(tf-gpu) jupyter notebook --ip $(ip addr show eno1 | grep 'inet ' | awk '{print $2}' | cut -d/ -f1) --no-browser
```

Jupyter notebook has to run in a compute node. That is why first need to allocate a compute node with ```srun``` command as shown in line 1 or 2 for a GPU or CPU node. Here, the server is lauched in line 5. Some critical output is printer, mainly the IP and port number, which should be annotated for using later to create the tunnel.

For the last part, open a local terminal in the user's PC/laptop, and enter the following instructions:

``` bash linenums="1"
cd /drives/c/Users/<local_user>/.ssh
ssh -p 55890 -i ./my-kelvin-key <user_name>@login.kelvin.alces.network -NL 8888:10.10.15.3:8888
```

In line 1, it is advisable to browse first to the folder where ssh Kelvin2 key is stored. This example uses Mobaxterm, for which drives locations start with "/drives/...". Line 2 stablishes the tunnel using ```ssh``` command, where it has been assumed that the IP and port number annotated above are "10.10.15.3" and "8888", respectively.

If not errors are reported during the execution of these commands, and the terminal looks like hanging out, then everything is ok, and the last step is to open a local browser and enter the adress "http://127.0.0.1 ...", which must have been shown in the output when the server connection was created.

## **R**

R is an open-source and free software which provides a programming language designed purposedly for statistical analyses. It is also highly preferable for data mining tasks and machine learning analysis. In Kelvin2 are available the following R's modules:

``` bash
apps/R/3.2.1/gcc-4.8.5+lapack-3.5.0+blas-3.6.0
apps/R/3.2.5/gcc-4.8.5+lapack-3.5.0+blas-3.6.0
apps/R/3.3.2/gcc-4.8.5+lapack-3.5.0+blas-3.6.0
apps/R/3.4.2/gcc-4.8.5+lapack-3.5.0+blas-3.6.0
apps/R/3.5.1/gcc-4.8.5+lapack-3.5.0+blas-3.6.0
apps/R/3.6.1/gcc-4.8.5+lapack-3.5.0+blas-3.6.0
apps/R/3.6.3/gcc-4.8.5+lapack-3.5.0+blas-3.6.0
apps/R/4.0.4/gcc-4.8.5+lapack-3.5.0+blas-3.6.0
apps/R/4.1.0/gcc-4.8.5+lapack-3.5.0+blas-3.6.0
apps/R/4.1.2/gcc-4.8.5+lapack-3.5.0+blas-3.6.0
apps/R/4.1.3/gcc-9.3.0+lapack-3.9.0+blas-3.8.0
apps/R/4.2.2/gcc-9.3.0+lapack-3.9.0+blas-3.8.0
apps/R/4.3.0/gcc-9.3.0+lapack-3.9.0+blas-3.8.0
```

By default, installing packages in R will fill up the users' quota (~50 GB hard disk + 100,000 files limit); therefore, it is advisable that the users setup an installation path for the packages. Below, the default install location is redirected to the user's scratch folder. The most transparent way to perform this operation is to create the file ".Renviron" in the home folder, which will keep the necessary R's environment variables to setup correctly the settings.

``` bash title="Setup R environment" linenums="1"
srun -p k2-hipri -N 1 -n 4 --time=3:00:00 --mem=16G --pty bash
mkdir /mnt/scratch2/users/$USER/R
mkdir /mnt/scratch2/users/$USER/R/lib
cd ~
echo "R_LIBS_USER=/mnt/scratch2/users/$USER/R/lib" > .Renviron
echo "R_LIBS=/mnt/scratch2/users/$USER/R/lib" >> .Renviron
echo "PKG_CONFIG_PATH=/mnt/scratch2/$USER/R/lib/pkgconfig" >> .Renviron
echo "" >> .Renviron
module load apps/R/4.3.0/gcc-9.3.0+lapack-3.9.0+blas-3.8.0
R
> .libPaths()
> quit()
```

In line 1, a compute node is requested to perform packages installation and work with R. Do not use login nodes to perform these operations. Lines 2-3 create the needed R's library folder. If they are not created, environment initialization may fail to be setup correctly. This operation needs to be done only one time at the very beginning of your R utilization in Kelvin2. It is neccesary to create ".Renviron" in the home folder which is guaranteed in line 4. Then, the ".Renviron" file is prepared in line 5-8. Notice, in line 8, this file must end with a newline character. If not, the instruction in the last line will be ignored without warning nor error. The environment file is also setup once, or every time the user wish to change the default install and library folders. Once the R's module is loaded and R is lauched in lines 9-10, the test of running the R's function ```.libPaths()``` must show correctly the new library path (line 11).

### ***Installing R's packages using an available module***

After setting up correctly the ".Renviron" file as presented above, installation of R's packages can proceed inside the R's application as follows:

``` bash linenums="1"
srun -p k2-hipri -N 1 -n 4 --time=3:00:00 --mem=16G --pty bash
module load libpng/16
module load apps/R/4.3.0/gcc-9.3.0+lapack-3.9.0+blas-3.8.0
export HDF5_USE_FILE_LOCKING='FALSE'
R
> if (!requireNamespace('BiocManager', quietly = TRUE))
> install.packages('BiocManager')
> BiocManager::install('DESeq2')
> installed.packages()
> quit()
```

Here, some issues may arise, such as that packages will not be installed correctly if some dependencies, libraries, or settings are not guaranteed. Therefore, the commands in lines 2, 4 are necessary in the present example. After that, the installation of the packages "BiocManager" and "DESeq2" is most straightforwardly done as shown in lines 7-8. Finally, the call to ```installed.packages()``` in line 9 will show all the installed R's packages.

When required libraries are not available in Kelvin2 module system, the users must contact the RSE Kelvin2 team. Otherwise, they can proceed to create an Anaconda environment, where installation of the R's preferred version, libraries and packages can be performed, as discussed in the next section.

## **R with Anaconda**

To create an environment on the user's scratch folder to install a particular version of R, follow these instructions:

``` bash title="R installation in an Anaconda environment" linenums="1"
srun -p k2-hipri -N 1 -n 4 --time=3:00:00 --mem=16G --pty bash
module load apps/anaconda3/2021.05/bin
export CONDA_PKGS_DIRS=/mnt/scratch2/users/$USER/conda/pkgs
export CONDA_ENVS_PATH=/mnt/scratch2/users/$USER/conda/envs
conda create -n R412env -c conda-forge r-base=4.1.2
```

Here, the preferred version (4.1.2) is specified in line 5 with the argument "r-base=4.1.2". For other versions, check the Anaconda R's online documentation, for example: search <https://anaconda.org/conda-forge/r-base> for the most recent version or <https://docs.anaconda.com/free/anaconda/packages/using-r-language/> for more information.

### ***Installing packages within an Anaconda environment***

The following example illustrates how to use this environment for installing a particular R's package (HOMER: <http://homer.ucsd.edu/homer/introduction/install.html>), proceeding from a clean/new Kelvin2 terminal connection.

``` bash
srun -p k2-hipri -N 1 -n 4 --time=3:00:00 --mem=16G --pty bash
module load apps/anaconda3/2021.05/bin
export CONDA_PKGS_DIRS=/mnt/scratch2/users/$USER/conda/pkgs
export CONDA_ENVS_PATH=/mnt/scratch2/users/$USER/conda/envs
source activate R412env
(R412env)$ conda config --add channels defaults
(R412env)$ conda config --add channels bioconda
(R412env)$ conda config --add channels conda-forge
(R412env)$ conda install -c bioconda samtools r-essentials bioconductor-deseq2 bioconductor-edger
(R412env)$ conda install homer
(R412env)$ conda update homer
(R412env) conda deactivate
```

Users should notice that when working with R inside an Anaconda environment, the libraries must be installed with the command ```conda install ...```. Whereas, the packages can be installed either in the same way by using conda install, or from the R's command line; for instance, using the function ```installed.packages()```, as shown in a section above. 

## **Ansys**

For more than 50 years, Ansys software has enabled innovators across industries to push boundaries with the predictive power of simulation. From sustainable transportation and advanced semiconductors, to satellite systems and life-saving medical devices, the next great leaps in human advancement will be powered by Ansys.
[https://www.ansys.com](https://www.ansys.com)

Ansys is a licensed software. In order to use it, users have to be registered in the license server. If you are not already included in the license server, you must contact the person in charge of it:

QUB: TBA </br>
Ulster: TBA

When the module is loaded, license parameters for QUB are loaded by default. Ulster users should change these license parameters in their batch script or interactive session. Contact the person in charge of the license for details.
User manual for Ansys is not publicly available, and only licensed users can access to it. To access to this material, register yourself in the Ansys web seite. </br>
[https://customercenter.ansys.com](https://customercenter.ansys.com){target=_blank}

### ***Load Ansys module***

The latest installed version on Kelvin-2 is the 2023R1. To load Ansys:

    $ module load apps/ansys/2023.1

### ***Ansys Fluent batch script example***

    #!/bin/bash

    #SBATCH --job-name=myfluentjob
    #SBATCH --output=myoutput.out
    #SBATCH --error=myerror.err
    #SBATCH --nodes=1
    #SBATCH --ntasks=32
    #SBATCH --partition=k2-hipri
    #SBATCH --mem=100G

    module load apps/ansys/2023.1

    ## QUB's license, already loaded with the module
    #export ANSYSLI_SERVERS=2325@143.117.212.118
    #export ANSYSLMD_LICENSE_FILE=1055@143.117.212.118

    ## Ulster's license
    #export ANSYSLI_SERVERS=2325@193.61.145.219
    #export ANSYSLMD_LICENSE_FILE=1055@193.61.145.219

    # Set architecture of the CPU (in this case amd64) and environment variables
    export FLUENT_ARCH=lnamd64
    export FL_TMPDIR=$SCRATCH/tmp

    # Create our hosts file 
    srun hostname -s | sort > hosts.$SLURM_JOB_ID.txt

    #Run Ansys Fluent.
    fluent 3ddp -g -t$SLURM_NTASKS -pinfiniband -mpi=openmpi -cnf=hosts.$SLURM_JOB_ID.txt -i my_fluent_input > my_fluent_output.res

## **Paraview**

A recommended method to run Paraview on Kelvin-2, it is to start a PV server on Kelvin-2, and to use your local machine to visualize the graphical interface. In this case, you need an installation of Paraview in your local machine as well.

With the following method, Paraview can be run in parallel, and use the local machine to work with the graphical interface.
It is necessary to set a tunnel through the Kelvin-2 login node.

As example, we are going to explain how to set the tunnel and connect with a computer outside the QUB campus. If the local machine is insithe the QUB campus, you have to remove the specification of the port `-p`, the identification private-key file `-i`, and to change the IP name to `kelvin2.qub.ac.uk`.

<ins>Step 1</ins>. Run pvserver in a computing node of Kelvin-2.

Open the session in Kelvin-2 as usual, and then open an interactive session. `<N>` will be the number of cores you require for your parallel job on Paraview.

    $ srun --pty --partition=k2-hipri --ntasks=<N> --mem-per-cpu=2G /bin/bash

Then, you are transferred to a computing node.

Now load the necessary modules, for paraview, and for OpenMPI.

    [<uid>@nodeNNN [kelvin2] ~]$ module load apps/paraview/5.8.1/bin
    [<uid>@nodeNNN [kelvin2] ~]$ module load mpi/openmpi/4.0.4/gcc-9.3.0+ucx-1.8.0

Run the server with the command

    [<uid>@nodeNNN [kelvin2] ~]$ mpirun -np <N> pvserver --force-offscreen-rendering --server-port=<YYYY>

Where `<uid>` is your Kelvin-2 username, `<N>` is the number of MPI tasks you want to run Paraview, and `<YYYY>` is a port number your choice.

<ins>Step 2</ins>. Create the tunnel to the port `<YYYY>`, through the port `<XXXX>` of your local machine.

In a different shell on your local machine, open the tunnel with the command:

    (local)$ ssh -L <XXXX>:nodeNNN:<YYYY> -p 55890 -i ~/.ssh/my-kelvin-key <uid>@login.kelvin.alces.network

Where `<uid>` is your Kelvin-2 username, `<YYYY>` is the port number you let open in the former step, and `<XXXX>` is a different port number your choice on your local machine.
`nodeNNN` is the computing node that the `srun` command directed you in the former step.

<ins>Step 3</ins>. Open paraview in your local machine.

Inside the application click:<br>
File - Connect<br>
- Give a name to the session, e. g. "remote machine"<br>
- In "Server Type" choose: Client / Server<br>
- host: localhost<br>
- Port: `<XXXX>`  (from step 2)<br>

Configure<br>
- Startup Type: Manual<br>
- Press Save<br>

Connect<br>
Once you created the client the first time, you can recover it the next time you open Paraview, you do not need to create it again, and you can connect directly in the step 3.

![Paraview Frontend](assets/Paraview_fronted.png)

## **Containers. Singularity.**

In Kelvin-2, we use singularity to run containers.
It has as main advantage, that it does not require root privileges to install the containers. Because of that, it is the mostly used container in HPC systems.

To activate singularity, one of the modules should be loaded

    apps/singularity/3.10.0
    apps/singularity/3.4.2

More detail about using singularity and containers on Kelvin-2 can be found in the 
[online seminar](https://gitlab.qub.ac.uk/qub_hpc/applications/-/tree/master/Singularity%20Seminar){target=blank}

### ***Running a Docker image on Singularity***

You need the docker image file. </br>
`myimage.img`</br>
As a tarball </br>
`mytarball.tar`

Go to a compute node

    $ srun --pty --partition=k2-hipri --ntasks=1 --mem-per-cpu=2G bash

Load the module

    $ module load apps/singularity/3.4.2

Convert the tarball to singularity:</br>
Firstly go to the directory where to tarball is located</br>

    $ cd /path/to/tarball

Convert the tarball:

    $ singularity build --sandbox mytarball docker-archive://mytarball.tar

Execute the image

    $ singularity shell myimage
    $ singularity exec myimage mycommand
    $ singularity run myimage


### ***Create the tarball from a Docker image***

These steps should be done in your local computer, where you have docker. Once you create the tarball, you have to copy it to Kelvin-2 and create the singularity image in Kelvin-2.

    <my_local_machine>$ docker save my_docker_image -o mytarball.tar

