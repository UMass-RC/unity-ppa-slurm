# unity-ppa-slurm
Slurm debian files for Unity PPA

### Setup
1. Download slurm archive and extract. Keep both the original archive and the extracted folder.
1. The archive should be named `slurm-wlm_20.02.7.orig.tar.gz`
1. The extracted folder should be named `slurm-wlm_20.02.7`
1. Clone this repository as "debian" inside the extracted folder
1. Install the following packages: `pbuilder` and `dh-make`
1. In your `~/.bashrc` or `~/.zshrc`, add these lines (replace with your own launchpad details)
   ```
   export DEBEMAIL="hakansaplakog@gmail.com"
   export DEBFULLNAME="Hakan Saplakoglu"
   ```
1. Create a pbuilder environment (in our case focal): `sudo pbuilder create --distribution focal`
