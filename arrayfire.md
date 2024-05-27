# Compiling PLUMED with ARRAYFIRE support
<p style="text-align: justify;">The purpose of this tutorial is to guide the user step by step through the process of compiling PLUMED with ARRAYFIRE, an open-source library that supports a wide range of hardware accelerators for parallel computing.</p>

<p style="text-align: justify;">In this specific case, the library is used to take advantage of CUDA-based GPUs to speed up the SAS calculation (ATOMISTIC / MARTINI / PARAMETERS / ONEBEAD representations). Furthermore, in order to show a limit case, the installation will be performed on Leonardo, an HPC hosted by CINECA and managed by the SLURM scheduler.</p>

<p style="text-align: justify;">In this specific scenario, the library is used to take advantage of CUDA-based GPUs to speed up the SAS calculation (ATOMISTIC / MARTINI / PARAMETERS / ONEBEAD representations). Furthermore, in order to show a real-life limit case, the installation will be performed on Leonardo, an HPC hosted by CINECA and managed by the SLURM scheduler.</p>

<p style="text-align: justify;">To keep the working environment clean and compatible with other setups, the modules are not automatically loaded via .bashrc/.bash_aliases, but are invoked during installation and then managed via the SBATCH script used to submit jobs on the HPC.</p>

## 1. Loading essential modules

<p style="text-align: justify;">To install ArrayFire, certain dependencies are required. Often, these dependencies are already available on HPC systems and can be loaded through modules. It is important to note that dependencies may also have their own dependencies, such as compilers or essential libraries. In general, to view the available modules on a SLURM-based HPC, you can use the command:</p>

``` module avail
```

```bash module avail
```

