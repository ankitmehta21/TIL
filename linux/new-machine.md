# Instantiating a new machine:
As Roberto explains in [this blog post](https://robertovitillo.com/2015/01/16/next-gen-data-analysis-framework-for-telemetry/), we have an easy way to generate spark clusters on AWS.
If I'm going to be using a cluster for any more than a command or two I usually add some configuration via my dotfiles.
I manage all local config through [homeshick](https://github.com/andsens/homeshick), which makes this really easy.

Here's the workflow once I tunnel into a freshly provisioned machine:
```
# Install homeshick
git clone git://github.com/andsens/homeshick.git $HOME/.homesick/repos/homeshick
printf '\nsource "$HOME/.homesick/repos/homeshick/homeshick.sh"' >> $HOME/.bashrc
source $HOME/.bashrc

# Save preconfigured .bashrc
mv ~/.bashrc ~/.bashrc_local

# Get my dotfiles and run my initialization script
homeshick -b clone git@bitbucket.org:harterrt/dotfiles.git &&
homeshick -b link dotfiles &&
sh ~/.init_new_machine.sh
```
