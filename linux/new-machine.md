# Instantiating a new machine:
As Roberto explains in [this blog post](), we have an easy way to generate spark clusters on AWS.
If I'm going to be using a cluster for any more than a command or two I usually add some configuration via my dotfiles.
I manage all local config through [homeshick](), which makes this really easy.

Here's the workflow once I tunnel into a freshly provisioned machine:
```
# Install homeshick
git clone git://github.com/andsens/homeshick.git $HOME/.homesick/repos/homeshick
printf '\nsource "$HOME/.homesick/repos/homeshick/homeshick.sh"' >> $HOME/.bashrc
source $HOME/.bashrc

# Get my dotfiles
homeshick clone https://harterrt@bitbucket.org/harterrt/dotfiles.git
homeshick link dotfiles
```
