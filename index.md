@def title = "Xuanzhao Gao's Site"
@def tags = ["syntax", "code"]

# Greetings!

I am Xuanzhao Gao, a PhD student in Hongkong University of Science and Technology.
This is my personal website, hope you enjoy!

# About Me

I am a PhD student at Hongkong University of Science and Technology (Guangzhou). My research interests include computational physics, application mathematics, and high-performance computing. I am currently working on the development of fast algorithms for kernel summation problems and tensor network contraction problems.

My Google Scholar profile can be found [here](https://scholar.google.fr/citations?user=ScbYSkgAAAAJ&hl=fr), and my github can be found [here](https://github.com/ArrogantGao).

# Blogs

1. [How to implement generic matrix multiplication (GEMM) with generic element types on GPU?](/blogs/CuTropicalGEMM/)

    This blog is a technical note for the Open Source Promotion Plan 2023 project ["TropicalGEMM on GPU"](https://summer-ospp.ac.cn/org/prodetail/23fec0105?lang=en&list=pro) released by JuliaCN, where I developed a [Julia](https://julialang.org/) package [CuTropicalGemm.jl](https://github.com/TensorBFS/CuTropicalGEMM.jl) calculate Generic Matrix Multiplication (GEMM) of Tropical Numbers on Nvidia GPUs.

2. [Tensor Network Contraction Order Optimization with Exact Tree Width Solver](/blogs/contractionorder/)

    This blog is a technical note for the [Google Summer of Code 2024](https://summerofcode.withgoogle.com) project ["Tensor network contraction order optimization and visualization"](https://summerofcode.withgoogle.com/programs/2024/projects/B8qSy9dO) released by [The Julia Language](https://julialang.org/), where I implemented an optimizer for tensor network contraction order based on tree decomposition in the Julia package [OMEinsumContracionOrders.jl](https://github.com/TensorBFS/OMEinsumContractionOrders.jl).

3. [Finding the Optimal Tree Decomposition with Minimal Treewidth](/blogs/treewidth/)

    This blog is a supplementary for the note [Tensor Network Contraction Order Optimization with Exact Tree Width Solver](/blogs/contractionorder/), where I detailed introduce the algorithm to find the optimal tree decomposition with minimal treewidth of a given simple graph, and how it is implemented in Julia package [TreeWidthSolver.jl](https://github.com/ArrogantGao/TreeWidthSolver.jl).

# Technical Notes

* [How to install slurm on Ubuntu 22.04](/blogs/slurm/)

  This blog is a technical note for the installation of slurm on Ubuntu 22.04 with NIS and apt tools.

# Open Source Packages

* [CuTropicalGEMM.jl](https://github.com/TensorBFS/CuTropicalGEMM.jl): A Julia package to speed up the generic matrix multiplication of Tropical Numbers using GPU.
* [ExTinyMD.jl](https://github.com/HPMolSim/ExTinyMD.jl): A simple but fast molecular dynamic framework in Julia.
* [ChebParticleMesh.jl](https://github.com/HPMolSim/ChebParticleMesh.jl): A Julia package to calculate the electrostatic potential of charged systems using Chebyshev interpolation (very similar to NUFFT).
* [TreeWidthSolver.jl](https://github.com/ArrogantGao/TreeWidthSolver.jl): A Julia package to solve tree decomposition with minimal treewidth of a given simple graph.
* [EwaldSummations.jl](https://github.com/HPMolSim/EwaldSummations.jl): A Julia package to calculate the electrostatic potential of charged systems using Ewald summation.
* [OptimalBranching.jl](https://github.com/ArrogantGao/OptimalBranching.jl): A Julia package to find the optimal branching rules for the maximum independent set problem.


# Publications

* Zecheng Gan, **Xuanzhao Gao**, Jiuyang Liang, and Zhenli Xu. [Fast Algorithm for Quasi-2D Coulomb Systems](https://doi.org/10.1016/j.jcp.2025.113733). Journal of Computational Physics, Volume 524, 1 March 2025, 113733

* **Xuanzhao Gao**, Xiaofeng Li, Jinguo Liu, and Zhenli Xu. [Programming guide for solving constraint satisfaction problems with tensor networks](https://arxiv.org/abs/2501.00227). arXiv, December, 2024.

* **Xuanzhao Gao**, Yi-Jia Wang, Pan Zhang, and Jin-Guo Liu. [Automated Discovery of Branching Rules with Optimal Complexity for the Maximum Independent Set Problem](https://arxiv.org/abs/2412.07685). arXiv, December, 2024.

* **Xuanzhao Gao**, Shidong Jiang, Jiuyang Liang, Zhenli Xu, and Qi Zhou. [A fast spectral sum-of-Gaussians method for electrostatic summation in quasi-2D systems](http://arxiv.org/abs/2412.04595). arXiv, December, 2024.

* Zecheng Gan, **Xuanzhao Gao**, Jiuyang Liang, and Zhenli Xu. [Random Batch Ewald Method for Dielectrically Confined Coulomb Systems](http://arxiv.org/abs/2405.06333). arXiv, May 10, 2024.

* Roa-Villescas Martin, **Xuanzhao Gao**, Sander Stuijk, Henk Corporaal, and Jin-Guo Liu. [Probabilistic Inference in the Era of Tensor Networks and Differential Programming](https://doi.org/10.1103/PhysRevResearch.6.033261.) Physical Review Research 6, no. 3 (September 6, 2024): 033261. 

* **XuanzhaoGao**, and Zecheng Gan. [Broken Symmetries in Quasi-2D Charged Systems via Negative Dielectric Confinement](https://doi.org/10.1063/5.0214523). The Journal of Chemical Physics 161, no. 1 (July 7, 2024): 011102.

# Talks

* [*Fast Summation Algorithms and Tensor Network Methods for Scientific Applications*](https://raw.githubusercontent.com/ArrogantGao/my_presentations/main/pre/sog_ob.pdf), Postdoc job talk at Flatiron Institute, Center for Computational Mathematics, online.
* [*How to Implement Generic Matrix-Mul with Generic Element Types on GPU?*](https://raw.githubusercontent.com/ArrogantGao/my_presentations/main/pre/CuTropicalGEMM.pdf), JuliaCN Meetup 2023, Shenzhen, China.
* [*A Fast Algorithm for Quasi-2D Coulomb systems*](https://raw.githubusercontent.com/ArrogantGao/my_presentations/main/pre/fast_algorithm_for_q2d_coulomb_systems.pdf), Scicade 2024, Tokyo, Japan.
* [*TreeWidthSolver.jl: From Treewidth to Tensor Network Contraction Order*](https://raw.githubusercontent.com/ArrogantGao/my_presentations/main/pre/treewidth.pdf), JuliaCN Meetup 2024, Guangzhou, China.