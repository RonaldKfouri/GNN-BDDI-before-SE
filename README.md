# Bad Data Detection and Identification Based on Graph Neural Network for Power System State Estimation

This repository contains example codes extracted from the paper "[Bad Data Detection and Identification Based on Graph Neural Network for Power System State Estimation](https://ieeexplore.ieee.org/document/11244249)", published in the _Journal of Modern Power Systems and Clean Energy_. 

The paper propses a Graph Neural Network (GNN) framework that detects and identifies bad data _before_ feeding the measurements into a state estimator. 

## Overview
A dataset is created. For each case, the load is varied according to pre-defined load curves, and the AC Power Flow problem is solved using Newton-Raphson method in MATPOWER. The results that converge are saved and Gaussian noise is added to emulate real-life scenarios. 

Then, a subset of measurements is selected as saved as node features and edge features. The outputs are encoded using a multi-class binary encoding scheme, as described in the paper. 

The code requires familiarity with MATLAB, MATPOWER, Python, PyTorch, PyTorch Geometric, and other famous ML libraries.

## Framework Workflow
1. [Load_Curves_14bus.md](Load_Curves_14bus.md) - Instructions on creating the loads; alternatively, you can download the `.mlx` file and run it directly in MATLAB;
2. [Generating_Data_for_Case14.md](Generating_Data_for_Case14.md) - Solving the AC Power Flow problem'
3. [Formulas_for_adding_Noise_Case14.md](Formulas_for_adding_Noise_Case14.md) - Adding synthetic Gaussian noise'
4. [Extracting_Feature_Names.md](Extracting_Feature_Names.md) - Extracting feature names (optional; you can use other tools);
5. Measurement selection - PMUs are placed on odd-indexed nodes and SCADA measurements are placed on even-indexed nodes. Measurement redundancy is guaranteed. You can use any library to prepare the measurements; they should be saved as three separate files: one for nodes features, one edge features, and one for the encoded outputs;  
6. [Fourtneen_Bus.ipynb](Fourtneen_Bus.ipynb) - GNN architecture and Edge-Conditioned Convolution (ECC) layer; also, for training and testing the algorithm;
7. [Explanations.ipynb](Explanations.ipynb) - GNNExplainer is extended and used to explain the feature importance of the nodes and edges. As with the paper, only the 6-bus case is shown.

----

**Notes**:
- The repository is structed to match the workflow described in the paper for full reproducibility.  
- Please cite the paper once it becomes available.

---

If you use this work in your research, please cite:

```bibtex
@ARTICLE{11244249,
  author={Kfouri, Ronald and Jabr, Rabih A. and Džafić, Izudin},
  journal={Journal of Modern Power Systems and Clean Energy}, 
  title={Bad Data Detection and Identification Based on Graph Neural Network for Power System State Estimation}, 
  year={2025},
  volume={},
  number={},
  pages={1-12},
  doi={10.35833/MPCE.2025.000178}}
```
