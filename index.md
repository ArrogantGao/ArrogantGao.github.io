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

    This blog is a technical note for the [Open Source Promotion Plan 2023](https://summer-ospp.ac.cn/) project ["TropicalGEMM on GPU"](https://summer-ospp.ac.cn/org/prodetail/23fec0105?lang=en&list=pro) released by JuliaCN, where I developed a [Julia](https://julialang.org/) package [CuTropicalGemm.jl](https://github.com/TensorBFS/CuTropicalGEMM.jl) calculate Generic Matrix Multiplication (GEMM) of Tropical Numbers on Nvidia GPUs.

2. [Tensor Network Contraction Order Optimization with Exact Tree Width Solver](/blogs/treewidth/)

    This blog is a technical note for the [Google Summer of Code 2024](https://summerofcode.withgoogle.com) project ["Tensor network contraction order optimization and visualization"](https://summerofcode.withgoogle.com/programs/2024/projects/B8qSy9dO) released by **The Julia Language**, where I developed a [Julia](https://julialang.org/) package [TreeWidthSolver.jl](https://github.com/ArrogantGao/TreeWidthSolver.jl) to calculate the tree decomposition with minimal treewidth of a given simple graph and made it a backend of [OMEinsumContracionOrders.jl](https://github.com/TensorBFS/OMEinsumContractionOrders.jl).

# Open Source Packages

* [CuTropicalGEMM.jl](https://github.com/TensorBFS/CuTropicalGEMM.jl)
  * A Julia package to speed up the generic matrix multiplication of Tropical Numbers using GPU.
* [ExTinyMD.jl](https://github.com/HPMolSim/ExTinyMD.jl)
  * A simple but fast molecular dynamic framework in Julia.
* [ChebParticleMesh.jl](https://github.com/HPMolSim/ChebParticleMesh.jl)
  * A Julia package to calculate the electrostatic potential of charged systems using Chebyshev interpolation (very similar to NUFFT).
* [TreeWidthSolver.jl](https://github.com/ArrogantGao/TreeWidthSolver.jl)
  * A Julia package to solve tree decomposition with minimal treewidth of a given simple graph.
  
# Publications

* **Gao, Xuanzhao**, and Zecheng Gan. [Broken Symmetries in Quasi-2D Charged Systems via Negative Dielectric Confinement](https://doi.org/10.1063/5.0214523). The Journal of Chemical Physics 161, no. 1 (July 7, 2024): 011102.

* Gan, Zecheng, **Xuanzhao Gao**, Jiuyang Liang, and Zhenli Xu. [Fast Algorithm for Quasi-2D Coulomb Systems](http://arxiv.org/abs/2403.01521). arXiv, March 3, 2024.

* Roa-Villescas, Martin, **Xuanzhao Gao**, Sander Stuijk, Henk Corporaal, and Jin-Guo Liu. [Probabilistic Inference in the Era of Tensor Networks and Differential Programming](http://arxiv.org/abs/2405.14060). arXiv, May 22, 2024.

* Gan, Zecheng, **Xuanzhao Gao**, Jiuyang Liang, and Zhenli Xu. [Random Batch Ewald Method for Dielectrically Confined Coulomb Systems](http://arxiv.org/abs/2405.06333). arXiv, May 10, 2024.