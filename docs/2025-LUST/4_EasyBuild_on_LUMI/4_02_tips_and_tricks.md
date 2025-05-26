# Tips & tricks

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

