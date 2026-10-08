-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Introduction
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

This code is a python3 jupyter notebook that simulates gravitational n-body systems using a 3D Velocity-Verlet symplectic integrator with Plummer softening (all calculations are done in a 3D Cartesian coordinate system) and provides numerical validations of the integrator using Kepler's 3rd Law, as well as numerical solutions to the 3-body problem and plots made using matplotlib to show how well the system's energy and angular momentum are conserved and how the simulations change with the timestep and softening constant they are run with. Please see the LaTeX report pdf for a detailed explanation of the code and the scientific results.

I wrote this code as one of MPhys Computational physics programming projects. There are two versions of this code in the repo - My original submission and an updated version. The updated version contains minor bug fixes, formatting changes and new, numerical solutions discovered in 2025 by X. Li and S. Liao, "Discovery of 10,059 new three-dimensional periodic orbits of general three-body problem", arXiv:2508.08568 (2025) that have movement in the Z axis as well as the X and Y axes, taking advantage of the 3D equations of motion in the code. 

The updated version also introduces a parallelised approach to validating Kepler's 3rd Law, enabling the 2-body simulations to be run in parallel. This yields a validation process that is not only much faster, but more statistically accurate since the Kepler constant can be calculated from a much larger pool of simulations in less time than it would take the older, single-threaded implementation to validate the Kepler constant for a smaller pool of simulations.

NOTE: The LaTeX report and feedback excel spreadsheet included in this repository are for my original submission. The updated version is something I made in my own time outside of studies. I have included pdf files of both fully executed notebooks in the repo since leaving the fully executed notebooks themselves there means they are too big to view on the GitHub website.

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Operating Systems and CPU architecture dependencies and requirements
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

Any version of Windows, macOS or Linux capable of running python3 will work. This code has been tested on x86 Windows, x86 Linux and ARM Apple Silicon CPU architectures. I haven't yet been able to test it on ARM Windows or ARM Linux, but if the python install is natively compiled for your CPU architecture and operating system there is a very good chance it will work just fine.

WARNING: It is not recommended to run this code through a python install that is running through an emulation or real-time translation layer such as Parallels, Apple Rosetta 2 or Wine since this will cause performance slowdowns or even numerical errors. The multiprocessing implementation used for the validation of Kepler's 3rd Law is demanding and will use all of the logical threads and CPU resources present on the CPU for running parallelised simulations, and some of the 3-body simulations run for up to 15 million timesteps. Please run through a python install that is natively compiled for your CPU architecture and operating system.

The sole exception to this rule is running it under WSL2 in Windows since this uses a genuine Linux image designed for the host CPU architecture (x86 and ARM supported).

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Python module dependencies (Minimum versions)
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

python >= 3.10.0 (recommended)

For python 3.10 and above:

NumPy >= 1.21.3
matplotlib >= 3.7.0
tqdm >= 4.21.0
Jupyter Notebook >= 7.0.0

Anaconda or miniconda are highly recommended, since these tools allow you to install jupyter notebook easily, and they are free.

WARNING: The .py multiprocessing worker file (parallelisable_integrator_functions.py) and all the .txt parameter files downloaded from GitHub must be present in the same directory / folder on your system for the code to work correctly. The code is designed to throw errors and stop running if it detects that the worker file or any of the parameter files are missing.

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Running the code
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

To run the code, please install Jupyter Notebook inside your python or conda environment.

Method 1: Use anaconda / miniconda, open the .ipynb file using Jupyter Notebook (Velocity_Verlet_Integrator.ipynb) and then run from the GUI: Kernel > Restart and run all cells.

Method 2: Alternatively, if you don't want to use anaconda / miniconda, use these commands to start jupyter and open the notebook:

cd /path/to/folder/containing/Velocity_Verlet_Integrator.ipynb
jupyter notebook Velocity_Verlet_Integrator.ipynb

Then run from the GUI: Kernel > Restart and run all cells.

Method 3: If you just want to run the notebook from the terminal without the GUI:

cd /path/to/folder/containing/Velocity_Verlet_Integrator.ipynb
jupyter execute --kernel_name=python3 Velocity_Verlet_Integrator.ipynb

NOTE: When the simulations have finished running, the plots of energy conservation, angular momentum conservation and the final orbit configuration are saved in the same directory the notebook and its parameter files are stored in.

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Velocity-Verlet Integration and Plummer Softening
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

The integrator uses the Velocity-Verlet algorithm, which is a symplectic integrator, to solve Newton's equations of motion for every body in 3D Cartesian coordinates. At each timestep the new positions are calculated from the old positions, velocities and accelerations, the accelerations are then recalculated from the new positions using Newton's law of gravitation, and finally the new velocities are calculated from the average of the old and new accelerations. Unlike leapfrog integration, the velocities are explicitly calculated and stored at every timestep, so the total energy and angular momentum can be calculated at exactly the same timesteps as the positions.

The gravitational acceleration of each body is calculated by summing over all of the other bodies, skipping the body itself to prevent a division by zero. Plummer softening (see equation 1 of Alexander Shirokov 2008, https://arxiv.org/pdf/0711.2989) is used, which adds the square of the softening constant (soft in the parameter files) to the squared distance between each pair of bodies. This stops the accelerations from diverging during close encounters between bodies. The same softened distance is used to calculate the gravitational potential energy (see equation 1 of Takayuki R. Saitoh and Junichiro Makino 2011, https://arxiv.org/pdf/1005.1752), so that the energy calculated by the code is consistent with the softened forces used to move the bodies and is correctly conserved.

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Kepler's 3rd Law Validation
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

The code validates the integrator using Kepler's 3rd Law, T^2 = 4*pi^2*R^3 / (G*(m1 + m2)), by running two-body simulations over a range of separations and comparing the numerically calculated orbital periods with the theoretical periods. Two different types of two-body system can be validated, chosen using the system_type parameter in the parameter file:

Star-planet: A circular orbit. The two bodies start on the x-axis on opposite sides of their common centre of mass, and are given the circular orbital velocities needed for their separation so that the centre of mass stays at rest.

Elliptical-binary: An elliptical orbit with an eccentricity of 0.5 between two bodies of equal mass (the code throws an error if the masses are not equal). The bodies start at their closest approach, with velocities calculated using the vis-viva equation. For these systems, R in Kepler's 3rd Law is the semi-major axis of the orbit rather than the starting distance between the two bodies.

For each simulation the separation is taken from an evenly spaced range between dist_min and dist_max, the theoretical period is calculated using Kepler's 3rd Law, the timestep is set to one ten-thousandth of the theoretical period and the simulation runs for two full orbits (20,000 timesteps). This is where multiprocessing comes in handy: The simulations are spread across all of the logical threads on the CPU using a pool of worker processes, with each thread running a single two-body simulation with a different separation until all of the num_tests simulations (128 in the supplied parameter files) are complete. The worker function used for this is imported from parallelisable_integrator_functions.py.

The numerical period of each simulation is calculated using the method described in the next section, and the results are plotted on three graphs: A graph of T^2 against R^3 comparing the numerical periods with the theoretical line from Kepler's 3rd Law, an agreement plot of the numerical periods against the theoretical periods, and an integrator accuracy plot showing the percentage error in the numerical period at each separation. The code also prints the percentage error between the numerically calculated Kepler constant (the mean of T^2/R^3 over all of the simulations) and the theoretical value of 4*pi^2 / (G*(m1 + m2)).

NOTE: If the period calculation fails for any of the simulations, those simulations are left out of the plots and the Kepler constant calculation, and a warning is printed telling you how many of the simulations failed. If it fails for all of the simulations, the code throws an error and stops.

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Calculating the Orbital Period
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

The orbital period is calculated by exploiting the fact that after every full period the bodies return to (very close to) their starting positions. At every timestep the code calculates the distance of each body from its starting position, and the period is the first timestep at which the largest of these distances drops below the period_tol tolerance set in the parameter file, multiplied by the size of the timestep. The first 5% of the timesteps are ignored so that the bodies are not detected as having returned before they have actually moved away from their starting positions.

If the bodies never return to within period_tol of their starting positions, no period is returned and the code prints "Period is None(s)". This is typically caused by a timestep or softening constant that is too large, or by one of the bodies being ejected from the system.

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Energy and Angular Momentum Conservation
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

Every simulation run using the run_integrator function calculates the total energy (the kinetic energy plus the softened gravitational potential energy) and the magnitude of the total angular momentum (the sum of the cross products of each body's position and momentum about the origin) of the system at every timestep. Both of these are plotted against the timestep to show how well the integrator conserves them, and a 3D plot of the trajectories of all of the bodies is made, with the final position of each body marked. Because the Velocity-Verlet algorithm is symplectic, the total energy should oscillate within a very small range rather than steadily drifting away from its starting value.

The median total energy, the median angular momentum and the period are printed at the end of each simulation. The median is used rather than the mean because it is resistant to anomalies in large datasets.

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Visualising the effect of changing the Timestep and Softening Constant
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

The next section of the code investigates how the simulations converge by re-running the figure of 8 3-body solution three times each (small, medium and large values) whilst changing the following parameters:

1. Timestep (0.001s, 0.01s and 0.1s, with a softening constant of 0.0001).
2. Softening constant (0.001, 0.01 and 0.1, with a timestep of 0.0001s).

Each of these simulations is 500,000 timesteps long. The energy conservation plots, angular momentum conservation plots, trajectories and periods can be compared with the standard figure of 8 simulation in the final section of the code, which uses both a timestep and a softening constant of 0.0001.

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Numerical Solutions to the 3-Body Problem
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

The final section of the code uses the integrator to produce numerical solutions to the 3-body problem for a selection of known periodic orbits. Each simulation reads its initial conditions from its own .txt parameter file, which contains the masses, initial positions and initial velocities of the three bodies, the gravitational constant G, the softening constant, the timestep, the number of timesteps, the period tolerance and the filename used to label the plots. The 2D (planar) solutions are:

1. Figure of 8.
2. Broucke A10.
3. Moth 1.
4. Butterfly 1.
5. Goggles.

The 3D solutions are periodic orbits found by X. Li and S. Liao (the O231(1.0) orbit is taken from X. Li and S. Liao, "Discovery of 10,059 new three-dimensional periodic orbits of general three-body problem", arXiv:2508.08568 (2025)):

1. O231(1.0) choreography.
2. O1.
3. O13.
4. O15.
5. O33.

Each of these simulations produces the energy conservation, angular momentum conservation and 3D trajectory plots described above, and prints the median total energy, median angular momentum and period of the system. 

