
# Fracture Parameters Calibration using Abaqus Simulations

This project code is used to optimize the Ductile Fracture Parameters in Extended modified Bai-Wierzbicki (eMBW) model

`Problem formulation`: Given an experimental force displacement (FD) curve of a geometry design (such as Notched Dog Bone NDBR50) of a material (such as DP1000) under a certain temperature (400 degrees), determine its Ductile Fracture Parameters such that when this parameters are fed as input to Abaqus, the software simulation will produce a fracture point on the FD curve that matches that of the experimental FD curve.    

`Workflow`: This project is a multiple objective optimization (MOO) task, which requires calibrating the Ductile Fracture Parameters to match fracture points of multiple FD curves (of different geometries) at once in the same temperature. 

`Stages`: This project has 2 main stages: producing a large number of initial guesses as the knowledge basis for optimization(Done using Latin Hypercube sampling), and running iterative calibration, where Bayesian optimization continues to update its surrogate model (Gaussian process) as more simulations are added to the knowledge basis.

## Status

![Build Status](https://img.shields.io/badge/build-passing-brightgreen.svg)
![Contributors](https://img.shields.io/github/contributors/springnuance/Abaqus-Macromechanics-Project.svg)
![Number of Commits](https://img.shields.io/github/commit-activity/y/springnuance/Abaqus-Macromechanics-Project.svg)

## Authors

- [@VedantZope](https://www.github.com/VedantZope). Tasks: Problem formulation, Optimization strategy, writing Abaqus python scripts, preparing simulation templates, running simulations and fine tuning results
- [@SpringNuance](https://www.github.com/springnuance) (Xuan Binh). Tasks: Optimization strategy, workflow design and programming project code. 

## Acknowledgements

 - [Professor Junhe Lian](https://scholar.google.com/citations?user=HO6x8pkAAAAJ&hl=en) for supervising this project
 - [Doctor Li Zinan](https://www.researchgate.net/profile/Zinan-Li-2) for providing experimental input data
 - [Doctor Rongfei Juan](https://www.researchgate.net/profile/Rongfei-Juan) for providing tips on presentation details

## How to run the project code

At the basics, these are the only folders and files required for running simulations. Others are not required or for postprocessing only. 

```
Abaqus-Macromechanics-Project
├───configs <- This is for global configurations. Users need to manually change the global_configs.xlsx
├───log <- Logging the progress into a text file
├───modules <- Containing python files as smaller modules that aid the calibration workflow
├───optimizers <- Containing Bayesian optimization code
├───paramInfo <- Defines Fracture parameter settings (range and exponent). Users need to manually define this
│   └───{material}
├───results <- Storing the simulation results
│   └───{material}
│       ├───{geometry}
│       │   ├───initial
│       │   │   ├───common
│       │   │   └───data
│       │   └───iteration
│       │       ├───common
│       │       └───data
├───simulations <- Temporary folders for storing simulations. After simulation, outputs are copied to result folder
│   └───{material}
│       ├───{geometry}
│       │   ├───initial
│       │   └───iteration
├───targets <- Storing experimental force-displacement curves that needs to be matched. Users need to manually add the FD curve excel files
│   ├───{material}
│   │   ├───{geometry}
├───stage0_configs.py <- Extracting the global configurations
├───stage0_initialize_directory.py <- Used to automatically initialize required project hierarchy folders
├───stage1_prepare_targetCurve.py <- Used to extract the FD curves from targets folder
├───stage2_run_initialSims.py <- Used to run initial guesses. Results are stored in results/{material}/{geometry}/initial
├───stage3_prepare_simCurves.py <- Used to extract initial guess simulations and iteration simulation (if they exist) and combine/interpolate the FD curves
├───stage4_iterative_calibration.py <- Used to run iterative parameter calibration. Results are stored in results/{material}/{geometry}/iteration
```

- Stage 1: Fixing the configs/global_configs.xlsx for your desire problem

- Stage 2: Running python stage0_initialize_directory.py. This is for folders generation

- Stage 3: You need to create FD_curve.csv under directory targets\{material}\{geometry}
         This csv file should have 3 columns name displacement,mm force,kN and force,N
- Stage 4: You need to create paramInfo.xlsx under directory paramInfo\{material}
         This csv file should have 4 columns depending on the number of parameters in Fracture model(eMBW In our case). Other columns like lowerBound, upperBound, exponent, name and unit should also be defined

- Stage 5: run the code
  ```
  $ pip install --user requirements.txt # Run only once
  $ python optimize.py # The only command you would need to run the project code
  ```
- Stage 6: The results will be output to the directory results\{material}\iteration
         Under this directory are iteration simulation result with index 1,2,3,... and a folder called common, which stores FD_Curves.npy
         The data structure of FD_Curves.npy is as follows:

  dictionary of \
  &nbsp;&nbsp;&nbsp;&nbsp; keys: parameter tuple of fracture model (("E1": value), ("E2": value), ("E3": value) ("E4": value)) \
  &nbsp;&nbsp;&nbsp;&nbsp; values: dictionary of force displacement \
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; key: force in N, value: numpy array \
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; key: displacement in mm, value: numpy array 

Note: The workflow is resistant to interruption. When you interrupted at iteration N, and when you rerun again with the same configurations, it will resume at iteration N again. 
         
- Stage 7: The workflow assumed that the initial guesses are wavy/jagged force-displacement curve.
         If FD curve produced by Abaqus is wavy, the code will use Savgol filter to smoothen the FD curves
