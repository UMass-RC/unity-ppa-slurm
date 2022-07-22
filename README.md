# Slurm Debian Packaging Files for PPA
Upstream PPA is [here](https://launchpad.net/~umass-rc/+archive/ubuntu/unity-cluster)

## Contributing

1. Prerequisites
   1. You must have a [launchpad](https://launchpad.net/) account and have GPG keys set up. For help with that, see [this](https://help.launchpad.net/Packaging/PPA).
      1. In addition, your system should be setup with this key in GnuPG.
   1. You must be a member of the [UMass RC Launchpad Group](https://launchpad.net/~umass-rc) and have write access
   1. You must have an ubuntu system (version doesn't matter), which can authenticate with GitHub using SSH.
1. Setup the environment on your ubuntu machine
   1. Install required deb pacakges
      1. `sudo apt install pbuilder dh-make`
   1. In your `~/.bashrc` or `~/.zshrc` file, add these lines (replace vars):

      ```
      export DEBEMAIL="<LAUNCHPAD EMAIL>"
      export DEBFULLNAME="<FULL LAUNCHPAD NAME>"
      ```
   1. Download Slurm release `wget https://github.com/SchedMD/slurm/archive/refs/tags/slurm-20-02-7-1.tar.gz`
      1. *Change the URL according to the release that is desired. Releases are [here](https://github.com/SchedMD/slurm/tags)*
   1. Extract using `tar -xf slurm-20-02-7-1.tar.gz`
   1. Rename the downloaded archive `mv <DOWNLOADED>.tar.gz slurm-wlm_<VERSION>.orig.tar.gz`
   1. Rename the extracted folder `mv <EXTRACTED FOLDER> slurm-wlm_<VERSION>`
   1. Inside `<EXTRACTED_FOLDER>`, clone this repository as `debian`
      1. `git clone git@github.com:UMass-RC/unity-ppa-slurm.git debian`
   1. Create a pbuilder environment (in our case focal): `sudo pbuilder create --distribution focal`
1. Making Changes
   1. **All changes made must remain inside of the debian folder.** Visit [here](https://www.debian.org/doc/manuals/maint-guide/start.en.html) for documentation on debian package files.
   1. After making changes, run `debchange` inside `<EXTRACTED FOLDER>`. This will launch a text editor with a change template in the changelog. Be sure to change `UNRELEASED` to `focal` and up the version number, and add any changes in the bullet points.
      1. The versioning follows Debian policies: the number before `ubuntu` is the debian version. If a debian version doesn't exist, use `0`. The number after is the incremental change for ubuntu. You should increment that number on any change.
   1. Run `pdebuild` inside `<EXTRACTED FOLDER>`. This will run `debuild`, but inside the chroot environment. This will also build this source into the deb packages specified.
   1. The result of `pbuilder` is in `/var/cache/pbuilder/result`. Take the deb packages there and install them on a focal system to make sure everything is working.
   1. In the same directory as the original source archive, use `debsign *.changes` for this.
   1. Finally, upload to ppa using `dput ppa:umass-rc/unity-cluster <source.changes>`, replace changes with the correct file in the parent dir.

## Changing Major Slurm Versions

There may come a time where we need to switch major slurm versions. For this, you will need to create a new branch in this repository, and start from the upstream debian source [here](https://salsa.debian.org/hpc-team/slurm-wlm/-/tree/debian/20.02.6-2).

On the debian upstream repository, find the branch that corresponds to the version desired and download it. This is the baseline for our new branch. Then, you have to make changes to these files to suit our needs. In our case, this *usually* entails:

* Removing `ConditionPathExists` from .service files for slurmdbd and slurmd since we use configless setup
* Removing files that match `*.init.d` and `*.default`
* Adding `--with-systemd` option to `rules` like so

   ```
   %:
	   dh $@ --builddirectory --with-systemd
   ```
* Adding a systemd section to the end of `rules`. This prevents services from auto enabling and starting on installation

   ```
   override_dh_installsystemd:
	   dh_installsystemd --no-start --no-enable
   ```

## Changing Major Ubuntu Release

If we are upgrading the cluster to a different ubuntu release, the package needs to be changed to reflect this. In *almost* all cases, this is as simple as adding a new changelog entry using `debchange`, except instead of `focal`, use the new release version (for example `jammy` for 22.04).

**IMPORTANT**: We have a testing PPA [here](https://launchpad.net/~umass-rc/+archive/ubuntu/unity-dev). I would **highly** recommend that any major version change or ubuntu release change be deployed on the testing PPA and tested on only a few nodes before widespread deployment.