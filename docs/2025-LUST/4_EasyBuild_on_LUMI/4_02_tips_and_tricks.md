# Tips & tricks

## Creating a full working copy of the software stack to develop in the core stack

This can be a lot of work, as it implies you'd also have to compile the toolchains you
want to use. For rare development, it may be better to use a copy in the LUST project (which
existed but was not maintained as noone used it).

Basic process:

-   Create a directory where the software stack will be installed. Call this `<DIR1>`

-   In that directory:

    ``` bash
    cd <DIR1>
    git clone git@github.com:Lumi-supercomputer/LUMI-SoftwareStack.git
    git clone git@github.com:Lumi-supercomputer/LUMI-EasyBuild-contrib.git
    # Next one not strictly needed unless you want to process documentation
    git clone git@github.com:Lumi-supercomputer/LUMI-EasyBuild-docs.git
    git clone git@github.com:Lumi-supercomputer/LUMI-EasyBuild-containers.git
    mkdir appl-local-containers
    cd appl-local-containers
    ln -s ../LUMI-EasyBuild-containers
    ln -s /appl/local/containers/sif-images
    ln -s /appl/local/containers/easybuild-sif-images
    cd -
    ```

    (And in fact, you may want to make a directory `easybuild-sif-images` instead and link to the individual
    files in the system as then you can add additional containers for testing without putting them already in
    `/appl/local/containers/easybuild-sif-images`.)

-   Also create a separate directory for your local user stack in which you want to experiment with
    building on top of the "central" stack in `<DIR1>`. Call this directory `<DIR2>`.

-   In that directory:

    ``` bash
    git clone git@github.com:Lumi-supercomputer/LUMI-EasyBuild-contrib.git
    ```

    and you could clone a repository with your personal easyconfigs also and call this `UserRepo`.

-   You need a script to set up the environment to use that copy of the software stack instead.
    E.g., with a bash function like

    ``` bash
    function init-lumi-h { # (Re-)initialize my personal LUMI test environment

        ###################################################################
        #
        # First part: Ensure our initialisation variables are set in case
        # init-lumi would be configured to preload an environment,
        #

        # Place of the installation
        # installroot="$HOME/LUMI"
        installroot="<DIR1>"

        # Our matching LUMI-user
        export EBU_USER_PREFIX="<DIR2>">"

        # Use the test container repository
        export LUMI_CONTAINER_REPOSITORY_ROOT='<DIR1>/appl-local-containers'

        # Force partition L as the others aren't ready yet
        # export LUMI_OVERWRITE_PARTITION='L'

        # LMOD to use
        installroot_lmod="/opt/cray/pe/lmod/lmod"


        ###################################################################
        #
        # Second part: LMOD initialisation
        #

        # Clear LMOD. We will restart it.
        # As it is a function we use eval.
        #eval 'clearLmod'
        eval 'module --force purge'
        eval $($LMOD_DIR/clearLMOD_cmd --shell bash --full)
        unset LUMI_INIT_FIRST_LOAD

        # Clear the lmod cache as we may be switching between versions of Lmod.
        [ -d $HOME/.lmod.d/.cache ] && /bin/rm -rf $HOME/.lmod.d/.cache  # System Lmod 8.3.1
        [ -d $HOME/.cache/lmod ]    && /bin/rm -rf $HOME/.cache/lmod     # Lmod 8.7.x

        # Resource the program environment initialisation
        # source /appl/lumi/LUMI-SoftwareStack/Setup/cray-pe-configuration.sh
        source /etc/cray-pe.d/cray-pe-configuration.sh

        # Correct the path in some variables read from the system file.
        sysroot='/appl/lumi'
        mpaths="${mpaths//$sysroot/$installroot}"
        LMOD_PACKAGE_PATH="${LMOD_PACKAGE_PATH/$sysroot/$installroot}"
        LMOD_RC="${LMOD_RC/$sysroot/$installroot}"
        LMOD_ADMIN_FILE="${LMOD_ADMIN_FILE/$sysroot/$installroot}"

        # Initialise LMOD
        source ${installroot_lmod}/init/profile

        # Build MODULEPATH
        mod_paths="/opt/cray/pe/lmod/modulefiles/core
                /opt/cray/pe/lmod/modulefiles/craype-targets/default
                $mpaths
                /opt/cray/modulefiles
                /opt/modulefiles"
        MODULEPATH=''
        for p in $(echo $mod_paths) ; do
            if [ -d $p ] ; then
                MODULEPATH=$MODULEPATH:$p
            fi
        done
        export MODULEPATH=${MODULEPATH/:/} # Export and remove the leading :.

        # Build LMOD_SYSTEM_DEFAULT_MODULES
        LMOD_SYSTEM_DEFAULT_MODULES=$(echo ${init_module_list:-PrgEnv-$default_prgenv} | sed "s_  *_:_g")
        export LMOD_SYSTEM_DEFAULT_MODULES
        # Need eval on the next line as it is a shell function.
        eval "module --initial_load --no_redirect restore"


        ###################################################################
        #
        # Third part: Personal finishing touches
        #

        # Set some aliases
        alias cdesr="cd    $installroot"
        alias pdesr="pushd $installroot"
        alias cdeur='cd    $EBU_USER_PREFIX'
        alias pdeur='pushd $EBU_USER_PREFIX'
        alias cdes="cd     $installroot/LUMI-SoftwareStack/easybuild/easyconfigs"
        alias pdes="pushd  $installroot/LUMI-SoftwareStack/easybuild/easyconfigs"
        alias cdec="cd     $installroot/LUMI-EasyBuild-contrib/easybuild/easyconfigs"
        alias pdec="pushd  $installroot/LUMI-EasyBuild-contrib/easybuild/easyconfigs"
        alias cdecc="cd    $LUMI_CONTAINER_REPOSITORY_ROOT/LUMI-EasyBuild-containers/easybuild/easyconfigs"
        alias pdecc="pushd $LUMI_CONTAINER_REPOSITORY_ROOT/LUMI-EasyBuild-containers/easybuild/easyconfigs"
        alias cdeu='cd     $EBU_USER_PREFIX/UserRepo/easybuild/easyconfigs'
        alias pdeu='pushd  $EBU_USER_PREFIX/UserRepo/easybuild/easyconfigs'
        alias upgrade-tc="$installroot/LUMI-SoftwareStack/tools/upgrade-tc.py"
        alias upgrade-locals="$installroot/LUMI-SoftwareStack/tools/upgrade-locals.lua"
        echo -e "\nAliases introduced by this command:\n" \
        "cdesr/pdesr    : system install root     : $installroot\n" \
        "cdeur/pdeur    : user install root       : $EBU_USER_PREFIX\n" \
        "cdes/pdes      : system EasyConfigs      : $installroot/LUMI-SoftwareStack/easybuild/easyconfigs\n" \
        "cdec/pdec      : contributed EasyConfigs : $installroot/LUMI-EasyBuild-contrib/easybuild/easyconfigs\n" \
        "cdecc/pdecc    : container EasyConfigs   : $LUMI_CONTAINER_REPOSITORY_ROOT/LUMI-EasyBuild-containers/easybuild/easyconfigs\n" \
        "cdeu/pdeu      : user EasyConfigs        : $EBU_USER_PREFIX/UserRepo/easybuild/easyconfig\n" \
        "upgrade-tc     : CSCS script to upgrade a toolchain in an EasyConfig\n" \
        "upgrade-locals : script to upgrade the local_*_version lines in an EasyConfig\n\n" \
        "Lmod version: $(eval 'module --version' |& grep Version | sed -e 's|.*Version *\(8.[[:digit:]]*.[[:digit:]]*\).*|\1|')\n"

    }
    ```

    but of course adapt `<DIR1>` and `<DIR2>`.

-   Now you can follow the steps in the ["Some procedures" page](https://lumi-supercomputer.github.io/LUMI-SoftwareStack/procedures/)
    of the [LUMI Software Stack technical documentation](https://lumi-supercomputer.github.io/LUMI-SoftwareStack/).

    This documentation is in fact generated from files in the `docs` subdirectory in the `LUMI-SoftwareStack` repository
    with a similar mkdocs environment as the LUMI Software Library.


## Long lists of elements for `preconfigopts` or options for `configopts`.

An example showing both is in recent [LUMI NAMD EasyConfigs](https://github.com/Lumi-supercomputer/LUMI-EasyBuild-contrib/tree/main/easybuild/easyconfigs/n/NAMD), e.g., 
["NAMD-3.0-cpeGNU-24.03-rocm-gpu-resident.eb"](https://github.com/Lumi-supercomputer/LUMI-EasyBuild-contrib/blob/main/easybuild/easyconfigs/n/NAMD/NAMD-3.0-cpeGNU-24.03-rocm-gpu-resident.eb).

-   [Line 100](https://github.com/Lumi-supercomputer/LUMI-EasyBuild-contrib/blob/main/easybuild/easyconfigs/n/NAMD/NAMD-3.0-cpeGNU-24.03-rocm-gpu-resident.eb#L100) shows `preconfigopts` 

    The example is not ideal as it would have been better to join with `&&` also to guarantee
    that the whole configure command fails if one of the commands in `preconfigopts` failed.

-   [Line 106](https://github.com/Lumi-supercomputer/LUMI-EasyBuild-contrib/blob/main/easybuild/easyconfigs/n/NAMD/NAMD-3.0-cpeGNU-24.03-rocm-gpu-resident.eb#L106)
    shows the `configopts`.

Of course, one could try to create one large string in Python, or break the assignment up, e.g.,

```
configopts  = '--charm-arch $EBTYPECHARMPLUSPLUS '
configopts += '--charm-base $EBROOTCHARMPLUSPLUS '
...
```

(and note the spaces that we now have to add at the end), but the "join" method has something
elegant, makes it very easy to add additional flags or commands, and is less error-prone.
There is also plenty of space left on each line to explain why an option is used or what it
means, to make the job easier for others who may want to update or customise this EasyConfig.


## Adding license information

EasyBuild has an EasyConfig parameter for that but it is rarely used in the regular EasyBuild repositories:

```
software_license_urls = [
    f'https://bitbucket.org/multicoreware/x265_git/src/{version}/COPYING',
]
```

One issue is that currently in our module scheme, it does nothing as the information is not being added to
the module.

We recently also started copying license information, etc., found in the sources of a package into 
`%(installdir)s/share/licenses/<name_of_package>` where for a bundle we use the name of each of the 
packages in the bundle for `<name_of_package>`. This is often easily done in `postinstallcmds` though 
for Bundle components it is easier to do so via `installopts` (adding the commands with the `&&` trick)
as there are no separate `postinstallcmds` for each Bundle component (at least, last time I tested those
did not work properly).

Some code fragments:

-   For software built with the `ConfigureMake` EasyBlock: As the build commands run in the sources directly,
    this will often work (but you may need to adapt the name of the files to copy):

    ```
    postinstallcmds = [
        'mkdir -p %(installdir)s/share/licenses/%(name)s',
        'cp COPYING %(installdir)s/share/licenses/%(name)s',   
    ]
    ```

-   With the `CMakeMake` EasyBlock, the build process runs in a separate directory, so you'll have to move
    to sources directory to copy:

    ```
    postinstallcmds = [
        'mkdir -p %(installdir)s/share/licenses/%(name)s',
        'cd ../%(namelower)s-%(version)s && cp AUTHORS CHANGELOG.md LICENSE.txt README.md README.SZIP THANKS %(installdir)s/share/licenses/%(name)s',   
    ]
    ```

    The `%(namelower)s-%(version)s` does not work for all software, you may have to check! E.g., you can just start EasyBuild but
    stop after the Prepare step with `--stop prepare` to inspect the sources.

-   The next one has worked for some `MesonNinja` software:

    ```
    postinstallcmds = [
        'mkdir -p %(installdir)s/share/licenses/%(name)s',
        'cd %(start_dir)s && cp AUTHORS CHANGELOG.md LICENSE.txt README.md README.SZIP THANKS %(installdir)s/share/licenses/%(name)s',   
    ]
    ```


## More to follow....

-   Static and shared libraries in CMakeMake packages (and using lib instead of lib64)

-   Fix Python shebang lines: EasyConfig parameter `fix_python_shebang_for`.  See the EasyConfigs for GLib.
  
    NOTE: There is currently only python3 on LUMI so this does not work as it should... So the GLib EasyConfigs for 24.03
    are wrong and will need a different trick.


*[[Next: Additional reading]](../5_00_additional_reading.md)*

