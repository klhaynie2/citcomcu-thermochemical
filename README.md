
         A Modified 3D Cartesian/Regional Spherical Citcom

                           Shijie Zhong
                         Dept of Physics
                      University of Colorado
                      Boulder, Colorado 80309

                    szhong@anquetil.colorado.edu

                      1. Background information.


This parallel 3D Cartesian/Regional spherical Citcom code was modified 
from the original Citcom code that was originally developed by L. Moresi
[Moresi and Gurnis, 1996]. The current version has the following
functionalities: modeling thermal convection and thermochemical convection
in either 3D Cartesian or 3D Regional spherical geometry. The current
version also considers the Boussinesq and extended-Boussinesq approximations
(i.e., with phase changes, adiabatic heating, viscous heating, and latent 
heating [Christensen and Yuen [1985]). Compared with the original muiltigrid
Citcom, the major new additions are: 1) parallel computing with MPI, 2) both
Cartesian and Regional Spherical geometry, 3) thermochemical convection 
with particles, 4) full-multigrid solver with consistent projection, 
and 5) extended Boussinesq approximations. For more information on the code
and benchmarks, see Zhong [2005a,b].

It is worthwhile to point out that there is an earlier version of Citcom that
also computes isochemical thermal convection for regional spherical geometry
[Billen et al., 2003] but with different implementation and formulation. 

The version of CitcomCU indluded in this repository was modified for instantaneous thermochemical convection without the use of tracers.

                      2. How to compile and run the code.

2.1. Compiling the code

In the src/ directory of this distribution, you will find 31 C source
files (*.c), 7 header files (*.h), and a makefile (Makefile). 

Before compiling the code, please make sure that your system is installed 
with MPI. You may need to edit the Makefile to specify the location of mpicc
and set various compiler flags for your specific machine. Then, to compile
the code and produce an executable citcom.mpi, type

	$ make citcomcu.mpi

2.2. Running the code

In the examples/ directory of this distribution, you will find several
input files. One of them (input1) is a sample input file. There are also
some input files (in the examples/Busse1993 directory) for benchmarks
against Busse et al. [1993].

You may first create directories for storing your output files, before 
running a calculation. Suppose that you use the sample input file input1,
you need to create directory CASE1 (see input1) in the directory from
which your code is going to run (often on a disk associated with the 
head node of your cluster). You also need to create directory CASE1 
on disks that are local to each compute node for this calculcation
(e.g., compute_node01, compute_node02, ..., compute_nodeXX). Most of
the output files are to be written to disks associated with compute
nodes. 

Then to run a calculation using citcomcu.mpi with input file input1, the
number of CPU NP, and machine file mc, type 

	$ mpirun -np NP -nolocal -machinefile mc citcomcu.mpi input1

where mc is a file listing processors to be used for the calculation. The 
above command running the code may be system-dependent, in particular,
the options -nolocal and -machinefile. You may need to discuss it with
your system administrator.

                      3. Getting to know the input file

The sample input file input1 provides some explanations for the various
input parameters. You may have to start with some simple calculations that
may only need a few essential parameters and then gradually learn other
functionalities of the code and their associated parameters.

The input file is divided into 10 sections which are briefly discussed
here.

Sect. 1. Input and Output Files Information.

The parameters in this section specify where the output and input files 
are. For example, the first parameter line

	datafile="CASE1/caseA"

specifies the directory and file head for all the output files.
Some output files, including log files, are stored in the directory
from which you run the calculation (often on the head node), and in
this example case, in subdirectory CASE1 as caseA.log0 and so on.
Most other output files (velocity or temperature) are stored in
disks on compute nodes in directory CASE1 with file head caseA
(e.g., caseA.ave.0.10000 and so on). Refer to Output.c to see
exactly what data is printed in the output files.

The second parameter "use_scratch" specifies whether output files 
are stored to compute nodes (recommended mode).

The next three parameters are about restarting a calculation for which
previous output, e.g., temperature, may be needed.

The next two important parameters "maxstep" and "storage_spacing" specify
the number of timesteps for the model run and output frequency. Here
output frequency for temperature and averaged properties (e.g., heat
flux) is often different. For details, check Output.c. You can modify
output in anyway you want.

Sect. 2. Geometry, Ra numbers, Internal heating, Thermochemical/Purely thermal convection.

The first two parameters specify the multigrid solver, which may never 
need to change. The next two parameters are Rayleigh number and compositional
Rayleigh number (their ratio is the buoyancy number). The compositional
Rayleigh number is only relevant if the next parameter "composition=1"
which turns on thermochemical convection.

The next two parameters "Q0" and "Q0_enriched" are nondimensional internal
heat generation rate for normal mantle and dense component of the mantle,
the latter is only relevant if "composition=1".

Parameters "markers_per_ele" and "comp_depth" specify the average number of
particles per element and initial depth of the density interface, again
both are only relevant if "composition=1".

The last two parameters "visc_heating" and "adi_heating" are switches for 
viscous heating and adiabatic heating that are implemented similar to
Christensen and Yuen [1985] for extended Boussinesq approximation.

Sect. 3. Grid And Multiprocessor Information

The first three parameters specify the number of processors used in
x (theta), z (r) and y (fi) directions. Of course, we assume that
the number of elements in each direction is fixed. The product of
three numbers is the total number of processors used for the calculation.

The next three parameters "nodex, nodey, and nodez" are numbers of 
nodes in each direction but ONLY relevant for conjugate gradient
solver.

The next four parameters specify the total number of elements for
the calculation with multigrid solver. The first three give the 
base level grid in each direction, and the fourth parameter "levels"
indicates how many times it gets doubled. For example, in the sample
input file input1, mgunitx=6,mgunitz=6,mgunity=6,levels=4, and the
total number of elements in each direction are 48, i.e., grid size
of 48x48x48. 

Note that number of processors in each direction is not entirely 
independent to the base level grid. More specifically, mgunitx must
be divisible by nprocx, and the same rule applies to y and z directions.

Sect. 4. Coordinate Information

The first parameter "Geometry" specifies which geometry you want for
the calculation, either 3D Cartesian or 3D regional spherical.
If 3D Cartesian is chosen, then the parameters for regional spherical
are irrelevant, and vice verse.

Here I will explain the setup for Cartesian cases, and regional spherical
parameters are similar.

For Cartesian setup, parameters "dimenx", "dimeny", and "dimenz" give
the box size in each direction. The next three parameters "z_grid_layers",
"zz" and "nz" are for coordinate information for z direction. 
"z_grid_layers" minus 1 is the number of layer in which the grid spacing
is uniform. "zz" and "nz" give the starting and ending z coordinates and 
nodal index. For the sample input, z_grid_layers=4, zz=0.0,0.1,0.9,1.0, 
nz=1,7,43,49, are for three grid layers that are bounded by coordinates
and nodal indices given in zz and nz, in z direction. The next six 
parameters are for x and y directions, respectively. In the sample file,
they give uniform spacing in those two directions.

The next three parameters "z_lmantle", "z_410" and "z_lith" are the 
nondimensional depth for 670-km, 410-km, and lithosphere. This information
may be used later in rheology definition.

Sect. 5. Rheology

The first parameter "rheol" specifies the type of rheological equation
that one may use. You need to check Viscosity_structures.c to see what are
available and it is very easy to implement your own. There are three 
rheological equations (options) available with this routine: rheol=0,1,or 2.
  * For rheol=0, eta = N0*exp[E(1-T)], where N0 is the pre-exponential 
    factor, and all are nondimensional.
  * For rheol=1, eta = N0*exp[E/(T+T_offset)], where T_offset is the 
    nondimensional surface temperature caused by nondimensionalization of 
    absolute temperature.
  * For rheol=2, eta = N0*exp{[E+(1-z)*V]/(T+T_offset)}, where (1-z) is the 
    nondimensional depth and V is nondimensional activation volume.
  * For rheol=3, eta = N0*exp[E(T_offset-T)].
  * For rheol=4, eta = N0*exp[E(T_offset-T)+(1-z)*V].

The second parameter "TDEPV" is a switch for temperature-dependent viscosity.
The parameter "VISC_UPDATE" is a switch for whether viscosity is to be updated.
It's clear that if "TDEPV" is on, then "VISC_UPDATE" should also be on.
The next parameter "update_every_steps" indicates how often viscosity is
to be updated (admittedly, this is a short-cut, but our experience seems
to suggest that viscosity and stiffness matrix do not have to be updated 
every time step).

The parameter "num_mat" specifies the number of material group that may
have different rheology (e.g., upper mantle, lower mantle, ...). In the
sample input file, num_mat=4 i.e., there can be four different material
groups. To see how they are defined, check routine construct_mat_group in 
Construct_arrays.c.

The next two parameters "visc0" and "viscE" are pre-exponential constants and
activation energy for each material group. In the sample input file, there
are four numbers for each parameter, and each number is for a material group.
The next two parameters "viscT" and "viscZ" can be defined for your own 
purposes. And check Viscosity_structures.c to see how you may use them.

Parameter "SDEPV" is a switch for non-Newtonian rheology, and "sdepv_misfit"
is the accuracy level for non-Newtonian iterations. Parameters "sdepv_expt"
and "sdepv_trns" are stress exponent and transition stress for each material
group. Non-Newtonian rheology is implemented in the code, but is rarely used.
So be careful with non-Newtonian rheology, if you decide to use it.

Parameters "VMIN" and "visc_min" specify whether or not a minimum viscosity
is imposed and what it is, if imposed. Likewise, "VMAX" and "visc_max" are
for the maximum viscosity.

Parameters "visc_smooth_cycles" and "Viscosity" do not need to vary.

Sect. 6. Dimensional information and depth-dependence.

The dimensional information is not really very useful except when 410-km
and 670-km phase changes are used (next section). 

Parameters "visc_factor" specifies a factor of viscosity increase with depth
as a linear function. This is in addition to whatever layered viscosity
structure we impose in section 5. In the sample input, "visc_factor=1" implies
that this linear increase with depth does not exist.

Parameters "thermexp_factor" and "thermdiff_factor" are the factors of thermal
expansion decrease and thermal diffusivity increase with depths as linear
functions.

Parameters "dispation_number" and "surf_temp" are dissipation number and non-
dimensional surface temperature. 

Sect. 7. Phase changes

Parameters "Ra_410" and "Ra_670" are the density drops at 410-km and 670-km
phase changes in SI unit. The next two parameters are the Clapeyron slopes.
Parameters "width410" and "width670" are the phase change widths (see
Christensen and Yuen [1985] for details).

You may need to check Phase_change.c to see how these dimensional numbers 
get nondimensionalized.

Sect. 8. Boundary conditions and initial perturbation

The default boundary conditions are free-slip on all sides. Periodic boundary
conditions are also implemented, however, you may want to talk to me before
using it.

Parameter "num_perturbations" specifies the number of perturbations of
different wavelengths. Parameters "perturbmag" and "perturbk" are the
magnitude and wavenumber of the perturbation. If you want, you may specify
perturbation for spherical harmonic order and degree for regional spherical
calculations with parameters "perturbl" and "perturbm". Initial fields are
specified in Convection.c (convection_initial_temperature). You should take
a look at this section of code, as they are often modified.

Sect. 9. Solver related matters.

Parameters in this section are rarely changed. Two parameters you may need to
be aware of are "accuracy" and "tole_compressibility", and they determine the
accuracy of the solutions.


                      4. Utilities

In the util/ direction of this distribution, there is a shell script to 
convert the output file to paralllel VTK files, which can be visualized
by several visualization packages, such as Paraview and LLNL VisIt. To
invoke the script, type

	$ citcomcu_write_vtk

to see the usage and arguments. 

The script has the following constraints:
  * The script can only convert files stored on local file system.
  * The script does not convert spherical coordinates to Cartesian
    coordinates.
  * The converted VTK files have their (x,y,z) coordinate axes mapped to the
    (z,x,y) axes, respectively, in CitcomCU data.


                      5. Acknowledgement and citation issues

The developers of this code have spent considerable amount of time in developing
and testing the code. We would appreciate if you can reference the following 
two papers for publications that result from using this code.

Zhong, S., Constraints on thermochemical convection of the mantle from plume
     heat flux, plume excess temperature and upper mantle temperature,
     J. Geophys. Res., 111, B04409, doi:10.1029/2005JB003972, 2006.

Moresi L.N. and M. Gurnis, Constraints on lateral strength of slabs
     from 3-D dynamic flow models, Earth Planet. Sci. Lett., 138, 15-28, 1996.


                      6. References

Billen, M.I., Gurnis, M., and Simons, M., Multiscale dynamic models of the
     Tonga-Kermadec subduction zone, Geophys. J. Int., 153, 359-388, 2003.
Busse, F.H. et al., 3D convection at infinite Prandtl number in Cartesian
     Geometry -- a benchmark comparison, Geophys. Astrophys. Fluid Dynamics,
     75, 39-59, 1993.
Christensen, U.R., and Yuen, D.A., Layered convection induced by phase changes,
     J. Geophys. Res., 90, 10,291-10,300, 1985.
Moresi L.N. and M. Gurnis, Constraints on lateral strength of slabs
     from 3-D dynamic flow models, Earth Planet. Sci. Lett., 138, 15-28, 1996.
Zhong, S., Constraints on thermochemical convection of the mantle from plume
     heat flux, plume excess temperature and upper mantle temperature,
     J. Geophys. Res., 2005a, in review. 
Zhong, S., Dynamics of thermal plumes in 3D isoviscous thermal convection, 
     Geophys. J. Int., 162, 289-300, 2005.

