* Introduction
* Installation

For the general installation instruction please refer to the
documentation of LAMMPS and to SPH-USER package:

- http://lammps.sandia.gov/doc/Section_start.html
- http://lammps.sandia.gov/doc/USER/sph/SPH_LAMMPS_userguide.pdf

** Installation examples
*** Ubuntu 12.04.5 LTS (Precise Pangolin)

- Install libraries
#+BEGIN_EXAMPLE
sudo apt-get install git
sudo apt-get install openmpi-dev
sudo apt-get install openmpi-bin
sudo apt-get install libjpeg-dev
#+END_EXAMPLE
- Clone the repository
#+BEGIN_EXAMPLE
git clone https://github.com/slitvinov/lammps-sph-multiphase.git lammps-sph
#+END_EXAMPLE
- Build the package
#+BEGIN_EXAMPLE
cd lammps-sph/src
make yes-USER-SPH
make linux CC=mpicc LINK=mpicc CCFLAGS='-O2 -g' FFT_LIB=-lm
#+END_EXAMPLE

*** Ubuntu 14.04.1 LTS (Trusty Tahr)

- Install libraries
#+BEGIN_EXAMPLE
sudo apt-get install git
sudo apt-get install g++
sudo apt-get install openmpi-bin
sudo apt-get install libopenmpi-dev
#+END_EXAMPLE
- Install libjpeg from source
#+BEGIN_EXAMPLE
wget http://www.ijg.org/files/
tar zxvf jpegsrc.v9a.tar.gz
cd jpeg-9a
./configure --prefix=${HOME}/prefix-jpeg-9a
make install
cd ..
#+END_EXAMPLE

- Clone the repository
#+BEGIN_EXAMPLE
git clone https://github.com/slitvinov/lammps-sph-multiphase.git lammps-sph
#+END_EXAMPLE
- Build the package
#+BEGIN_EXAMPLE
cd lammps-sph/src
make yes-USER-SPH
make linux CC=mpic++ LINK=mpic++ CCFLAGS="-O2 -g -I${HOME}/prefix-jpeg-9a/include" FFT_LIB=-lm LINKFLAGS="-O -L${HOME}/prefix-jpeg-9a/lib" MPI_LIB=
#+END_EXAMPLE
*** CentOS Linux 6.5

- Install libraries
#+BEGIN_EXAMPLE
sudo yum install git
sudo yum install mpich2-devel
sudo yum install gcc-c++
sudo yum install libjpeg-devel
#+END_EXAMPLE
- Clone the repository
#+BEGIN_EXAMPLE
git clone https://github.com/slitvinov/lammps-sph-multiphase.git lammps-sph
#+END_EXAMPLE
- Build the package
#+BEGIN_EXAMPLE
cd lammps-sph/src
make yes-USER-SPH
make linux CC=mpicc LINK=mpicc CCFLAGS='-O2 -g' FFT_LIB=-lm MPI_LIB=
#+END_EXAMPLE

*** Debian Wheezy 7.5 x64

- Install libraries
#+BEGIN_EXAMPLE
sudo apt-get install mpich2
sudo apt-get install libjpeg8-dev
#+END_EXAMPLE
- Clone the repository
#+BEGIN_EXAMPLE
git clone https://github.com/slitvinov/lammps-sph-multiphase.git lammps-sph
#+END_EXAMPLE
- Build the package
#+BEGIN_EXAMPLE
cd lammps-sph/src
make yes-USER-SPH
make linux CC=mpicc LINK=mpicc CCFLAGS='-O2 -g' FFT_LIB=-lm MPI_LIB=
#+END_EXAMPLE

* Installation using src/Make.py

#+BEGIN_EXAMPLE
cd    src
mkdir -p MAKE/MINE
python Make.py -o linux  -v -p user-sph  -cc mpi -fft none -jpg no -a file mpi
#+END_EXAMPLE

* Implementation
We add the following extension to USER-SPH package:
** atom_style meso/multiphase
This is data structures which provides
- position
- velocity
- extrapolated velocity (=vest=)
- forces
- SPH density (=rho=)
- time derivative of SPH density (=drho=)
- internal energy per particle (=e=)
- time derivative of internal energy per particle (=de=)
- color gradient vector (=colorgradient=)
- per-particle heat capacity (=cv=)

This data structure can be activated by
#+BEGIN_EXAMPLE
atom_style meso/multiphase
#+END_EXAMPLE

** pair_sph_colorgradient
A [[http://lammps.sandia.gov/doc/pair_style.html][pair_style]] to calculate a color gradient
#+BEGIN_EXAMPLE
pair_style         sph/colorgradient
pair_coeff         I J     ${h} ${alpha}
#+END_EXAMPLE
Here, =I= and =J= are the types of SPH particles for which a color
gradient is calculated, =alpha= is a surface tension coefficient, =h=
is a cutoff.

** pair_sph_surfacetension
A [[http://lammps.sandia.gov/doc/pair_style.html][pair_style]] to calculate surface tension

#+BEGIN_EXAMPLE
pair_coeff         I J     sph/surfacetension ${h}
#+END_EXAMPLE

Here, =I= and =J= are the types of SPH particles for which a surface
tension is calculated, =h= is a cutoff. Note that surface tension
coefficient is included into color gradient.

** pair_sph_heatconduction_phasechange
A modified heat conduction equation to use for phase change model. Has
to forms. Simple form is equivalent to the heat conduction equation
from USER-SPH package
#+BEGIN_EXAMPLE
pair_coeff         I J  sph/heatconduction/phasechange  ${D_heat_ld}
#+END_EXAMPLE
Here, =I= and =J= are the types of SPH particles which interact and
=D= is a heat diffusion coefficient.

Full form of the pair style is
#+BEGIN_EXAMPLE
pair_coeff         I J  sph/heatconduction/phasechange  ${D_heat_ld} TI TJ
#+END_EXAMPLE
where =TI= and =TJ= are temperatures for corresponding particles in
=I= and =J= interactions.

=NULL= can be used as a placeholder to indicate that normal temperate
should be used for corresponding particle
#+BEGIN_EXAMPLE
pair_coeff         I J  sph/heatconduction/phasechange  ${D_heat_ld} TI NULL
#+END_EXAMPLE

** fix_phase_change
Fix which adds a phase change
#+BEGIN_EXAMPLE
fix                fix_ID group_ID phase_change &
                   ${Tc} ${Tt} ${Hwv} ${dr} ${mass_v} &
		                      ${pcutoff} ${l_type} ${v_type} ${insert_every} 123456 ${prob} region
				      #+END_EXAMPLE
				      =fix_ID= and =group_ID= are described in LAMMPS documentation. =TC= is
				      critical temperature of the phase change, =TT= is transition
				      temperature for the algorithm (should be set above =TC=), =dr= a
				      characteristic distance for a new particle position, =mass= a mass of
				      a new particle, =h= cutoff of the interaction, =from_type= and
				      =to_type= types of the particles involved in phase transition, =N=
				      frequency of the check for phase transition algorithm, =seed= a seed
				      for random number generator, =prob= probability of the phase
				      transition if all criteria are met (=0<prob<1=), =region= a region
				      where algorithm checks for potential phase transition.

* Examples
See [[file:examples/USER/sph/]]