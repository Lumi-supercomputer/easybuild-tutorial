# Jülich Supercomputing Centre

(*author: Alan O'Cais, Jülich Supercomputing Centre)*

## General info

<img src="../../img/jsc.jpg" style="float:right" width="40%"/>

The Jülich Supercomputing Centre
(JSC, [https://www.fz-juelich.de/ias/jsc](https://www.fz-juelich.de/ias/jsc/EN)) at 
Forschungszentrum Jülich has been operating the first German supercomputing centre since
1987, and with the Jülich Institute for Advanced Simulation it is continuing the long
tradition of scientific computing at Jülich. JSC operates one of the most powerful
supercomputers in Europe (JUWELS), and computing time at the highest performance
level is made available to researchers in Germany and Europe by means of an independent
peer-review process.

### Staff & user base

About 200 experts and contacts for all aspects of supercomputing and simulation sciences
work at JSC. JSC's research and development concentrates on mathematical modelling and
numerical simulation, especially parallel algorithms for quantum chemistry, molecular
dynamics and
Monte-Carlo simulations. The focus in the computer sciences is on cluster computing,
performance analysis of parallel programs, visualization, computational steering and
federated data services.

In cooperation with hardware and software vendors like IBM, Intel and ParTec,
JSC meets the challenges that arise from the development of exaflop systems - the
computers of the next supercomputer generation. As a member of the German Gauss Centre
for Supercomputing, JSC has also coordinated the construction
of the European research infrastructure "PRACE - Partnership for Advanced Computing in
Europe" since 2008.

### Resources

JSC currently manages 3 primary systems (in addition to a
number of other development clusters):

[JUWELS](https://www.fz-juelich.de/ias/jsc/EN/Expertise/Supercomputers/JUWELS/Configuration/Configuration_node.html)
is a milestone on the road to a new generation of ultra-flexible modular supercomputers
targeting a broader range of tasks. It currently has 10.6 (CPU) + 1.7 (GPU) Petaflop per
second peak performance. In the coming months, JUWELS will become the first
supercomputer equipped with NVIDIA A100 GPU, when a new GPU module providing an
additional 58 Petaflops will be installed. This module will make JUWELS the most
powerful supercomputer in Europe.

[JURECA](https://www.fz-juelich.de/ias/jsc/EN/Expertise/Supercomputers/JURECA/Configuration/Configuration_node.html)
is the precursor system to JUWELS with 1.8 (CPU) + 0.44 (GPU) + 5 (KNL) Petaflop per
second peak performance. It is due to be decommissioned at the end of November 2020. The
technical details about the successor system, the JURECA DC (data centric) module,
will be announced soon.

[JUSUF](https://www.fz-juelich.de/ias/jsc/EN/Expertise/Supercomputers/JUSUF/Configuration/Configuration_node.html)
combines an HPC cluster and a cloud platform in a single system with homogeneous
hardware such that resources can be flexibly shifted between the partitions. The JUSUF
compute nodes are equipped with two AMD EPYC Rome CPUs, each with 64 cores. One third of
the compute nodes are furthermore equipped with one NVIDIA V100
GPU. The JUSUF cluster partition will provide HPC resources for interactive workloads
and batch jobs. The cloud partition will enable co-location of (web) services with these
resources to enable new workflows and support community platforms.

## Usage of EasyBuild within JSC

As a large site with multiple systems and diverse requirements, JSC takes advantage of
how easily EasyBuild
can be extensively configured according to site policies, ranging from the software
installation prefix to
all aspects of the module naming scheme being used for the modules being generated.

JSC maintains a [public repository of the customisations and development environment
for EasyBuild](https://github.com/easybuilders/JSC) that we use in our production
environment. Below we highlight some particular cases of these customisations.

### Custom toolchains

As of June 2020,
there are a total of 15 unique toolchain definitions in use at JSC, which reflect
multiple
combinations of compilers (`GCCcore`, `GCC`, `Intel` and `PGI`),
MPI runtimes (`ParaStationMPI`, `OpenMPI`, `IntelMPI` and `MVAPICH2`)
and mathematical libraries (`MKL`).

Given the proliferation of toolchains required at our site, JSC has put a lot of effort
into increasing the capabilities of the `--try-toolchain` option and has recently
introduced the `--try-update-deps` experimental option to more easily adopt upstream changes and adapt them to our
environment. 

### Custom module naming scheme

By default
EasyBuild includes both the flat and hierarchical module naming schemes
and these can be leveraged as examples for custom schemes.
JSC employs such a
[custom scheme (based closely on the standard hierarchical
scheme)](https://github.com/easybuilders/JSC/blob/master/Custom_MNS/2019a/flexible_custom_hierarchical_mns.py)
to control the exact structure of the
hierarchy and the naming of some specific modules (such as
the compilers and MPI runtimes).

### Custom easyconfigs

The upgrade cycle for our software stack does not exactly match that of EasyBuild (see
[below](#upgrading-and-retiring-software) for context on this). This means that the
versions of software and dependencies that we provide may be slightly different than
what is in the main repository (due to critical updates, releases of important software,
etc.). Arising from this and the
custom toolchains that we use, we maintain our own reference easyconfig repository (our
*Golden* repository).

We are actively trying to minimise the differences between the two (see our
[usage of hooks](#usage-of-hooks) below) as we recognise that this
introduces an additional maintenance burden for us, and inhibits our ability to
easily contribute back our easyconfigs to EasyBuild.

### Hiding dependencies

While we provide an extensive set of software, we try to minimize the packages exposed
to the users by hiding a large set of dependencies which users
are unlikely to require directly (via the `hide-deps` configuration setting). There are currently
over 200 such hidden dependencies.  

While hidden dependencies are not visible in the `module` view by default, users can
expose them by the use of the `--show-hidden` argument in `Lmod`:
```shell
module --show-hidden avail
``` 

### Usage of hooks

The relatively new [***hooks*** feature](https://docs.easybuild.io/en/latest/Hooks.html)
of EasyBuild provides JSC with an opportunity to
track upstream developments more closely.

We are currently integrating a new hook that provides a lot of useful functionality:

* Facilitates ***userspace installations*** alongside system provided installations
  * Restricts users from installing non-supported compilers (in particular we don't want
    people to install their own `GCCcore` since this would likely lead to an avalanche
    of required dependencies) and MPI runtimes (since MPI installations
    are heavily customised)
  * Restricts users to only resolve dependencies from our *Golden* repository (as well
    as from
    their own installed software) but allows them to search in the upstream repositories
    * if they try to install something from the upstream repository, the hook advises
      them how to do this *correctly* for our systems
* Customises the final module files
  * Customises the names of some modules (such as `Intel` over `iccifort` and
    `IntelMPI` over `impi`)
  * Injects an Lmod *family* in the modules of our compilers and MPI runtimes
  * Adds Lmod *properties* for GPU enabled applications and user installed software so
    that they can be easily identified in the `module` view
  * Adds a `site_contact` for all modules
* Updates the Lmod cache when an installation is made system-wide

We see potential in the use of hooks as a great way of encouraging, documenting and
automating "correct" installation processes for our system.

### Upgrading and retiring software

The expected lifetime of a system like JURECA is roughly five years.
Within that period one can expect updates to compilers every
few months and updates to MPI implementations as the latest
standards are integrated. This would mean that the entire
software stack will require frequent upgrades. During such
upgrades it is natural to expect that one would install the latest
version of any particular software package.

The project cycles at JSC lasts 12 months with two
cycles per year. When new users get access to the machine,
we want them to only be exposed to the latest software with the
latest compilers. For this reason, we have chosen six months
as our upgrade period and we chose to retire outdated software
versions with the same frequency. We call these software
upgrades "stages". For each 'stage', we select the toolchains that
we will support and rebuild the latest versions of our supported
software with these toolchains. We chose a prototype toolchain
as a template and, once fully populated, migrate the changes
to our other toolchains.

We expect members of the support team to contribute to
software installations since it is common that application
software requires specific knowledge to be installed and tested
appropriately. We provide a special development stage with the
latest toolchains for the support team where they can prepare
their easyconfig files for inclusion in the upgrade. Once a
software package has been successfully built and tested, it is
added to a *Golden* repository to be used for the stage upgrade.

The default stage visible to users is controlled by a symbolic
link. Stage upgrades are prepared in a separate environment
to this default. Once the upgrade has been implemented, users
are given three weeks notice and the symbolic link is updated
during a maintenance window. Users are provided with the
capability of continuing to use a retired stage if they wish
to do so. However, additional software requests are (typically)
only accepted for the current default stage.

While stage upgrades may introduce some overhead for
existing users (they may need to recompile their code and
modules may be named differently in particular cases), there
are clear benefits to using the latest compilers and software
stack. In addition, these upgrades provide us with the opportunity
to potentially change our module hierarchy or introduce
new features related to Lmod.
