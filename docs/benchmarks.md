# Benchmarks

This page contains benchmarking results for the Kelvin2 system.

## Intel Data Center GPU Max 1100

This section shows results of a performance comparison between Intel and NVidia GPUs on the Kelvin2 HPC system. The training scripts may be found [here](https://github.com/stevencousens/kelvin2-gpu-performance-tests){target=_blank}

### Benchmark 1: Matrix Multiply (PyTorch)

#### Method
- Multiply two 8192x8192 matrices using PyTorch
- Run multiple iterations and measure execution time
- Convert throughput to TFLOPS

#### Results

<p align="center">
<img src="/nihpc-documentation/assets/benchmarking/matmul-results-1.png" alt="Matrix Multiply Results" height="600">
</p>

### Benchmark 2: Convolutional Neural Network (PyTorch)

#### Method

- Run a standard ResNet‑50 training loop on each GPU
- Use synthetic image data created directly on the GPU
- Measure end‑to‑end training throughput (images/sec)
- Use automatic mixed precision (AMP)

#### Results
<p align="center">
  <img src="/nihpc-documentation/assets/benchmarking/cnn-results-1.png" alt="CNN Results" height="300">
</p>

### Benchmark 3: Molecular Dynamics Simulation (GROMACS)

#### Method

- Use a standard benchmark from [https://www.mpinat.mpg.de/grubmueller/bench](https://www.mpinat.mpg.de/grubmueller/bench)
- BenchPEP-h: 12M atoms, Peptides in water, 2 fs time step, h-bonds constrained

#### Results

<p align="center">
  <img src="/nihpc-documentation/assets/benchmarking/gromacs-results-1.png" alt="GROMACS BenchPEP-h results" height="300">
</p>
