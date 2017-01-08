openclerk-vagrant
=================

A Vagrantfile for setting up a new Vagrant box with Openclerk installed,
using [openclerk-chef solo setup](https://github.com/soundasleep/openclerk-chef)
(which then uses the [openclerk-cookbook](https://github.com/soundasleep/openclerk-cookbook)).

# Installing

Install Vagrant (and VirtualBox as necessary), and then run `vagrant up`.

To apply any changes made to `Vagrantfile`, run `vagrant provision` to reprovision everything.

# Done so far

1. Install rvm, chef-solo, librarian-chef
1. Clone [openclerk-chef](https://github.com/soundasleep/openclerk-chef)
1. Copy [web.json](web.json) to the local /chef-repo directory, so we can configure
   Openclerk from outside Vagrant
1. Run `chef-solo` and start getting things installed

# Developing

Anything in `local-repo/` will be the same as the Vagrant chef repository,
so you can use your IDE of choice to edit, etc.

### Update `Cheffile.lock`, run `chef-solo`, etc:

```
vagrant ssh
sudo passwd root        # reset the root password
su
cd /root/chef-repo
```

### Update single cookbooks directly:

```
cd /root/chef-repo/cookbooks/chef_bundler
git commit -p -v
```

# TODO

1. Get `chef-solo` to execute successfully and actually get an Apache running
   (it looks like openclerk-cookbook is not complete yet)
1. Docs on how to actually run chef-solo outside of `vagrant provision`
1. Put all of the inline scripts into separate `.sh` files. Better log info.
