The only command you would need to run the project code is 
python optimize.py

Stage 1: Fixing the configs/global_configs.xlsx for your desire problem
Stage 2: Running python stage0_initialize_directory.py. This is for folders generation
Stage 3: You need to create FD_curve.csv under directory SOO_targets\{material}_{hardeningLaw}_{geometry}_curve{curveIndex}
         This csv file should have 3 columns name displacement,mm force,kN and force,N
Stage 4: You need to create paramInfo.xlsx under directory SOO_paramInfo\{material}_{hardeningLaw}_{geometry}_curve{curveIndex}
         This csv file should have N columns depending on the number of parameters in hardening law. Other columns like lowerBound, upperBound, exponent, name and unit should also be defined
Stage 5: Drag the whole project code onto CSC and run
         cd projectDirectory
         module load python-data
         pip install --user requirements.txt
         python optimize.py
Stage 6: The results will be output to the directory SOO_results\{material}_{hardeningLaw}_{geometry}_curve{curveIndex}\iteration
         Under this directory are iteration simulation result with index 1,2,3,... and a folder called common, which stores FD_Curves.npy and flowCurves.npy. 
         The data structure of FD_Curves.npy is as follows:
         dictionary of 
          keys: parameter tuple of hardening law (("c1": value), ("c2": value), ("c3": value))
          values: dictionary of force displacement
             key: force in N, value: numpy array
             key: displacement in mm, value: numpy array
        
        The data structure of flowCurves.npy is as follows:
         dictionary of 
          keys: parameter tuple of hardening law (("c1": value), ("c2": value), ("c3": value))
          values: dictionary of force displacement
             key: stress in MPa, value: numpy array
             key: strain in -, value: numpy array

Note: The workflow is resistant to interruption. When you interrupted at iteration N, and when you rerun again with the same configurations, it will resume at iteration N again. 
         
Stage 7: The workflow already assumed that the initial guesses are smooth force-displacement curve.
         If FD curve produced by Abaqus is wavy, please use notebooks/smooth.ipynb to process the FD curves