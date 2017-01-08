# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://atlas.hashicorp.com/search.
  config.vm.box = "bento/ubuntu-16.04"

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  config.vm.network "forwarded_port", guest: 80, host: 8080

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # config.vm.network "private_network", ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  # config.vm.provider "virtualbox" do |vb|
  #   # Display the VirtualBox GUI when booting the machine
  #   vb.gui = true
  #
  #   # Customize the amount of memory on the VM:
  #   vb.memory = "1024"
  # end
  #
  # View the documentation for the provider you are using for more
  # information on available options.

  # Let's share a common cache omg
  if Vagrant.has_plugin?("vagrant-cachier")
    # Configure cached packages to be shared between instances of the same base box.
    # More info on http://fgrehm.viewdocs.io/vagrant-cachier/usage
    config.cache.scope = :box
  else
    fail "You should have the vagrant-cachier plugin installed to improve your quality of life. Try: `vagrant plugin install vagrant-cachier`"
  end

  # Stop 'mesg: ttyname failed: Inappropriate ioctl for device'
  # https://github.com/mitchellh/vagrant/issues/1673
  config.ssh.shell = "bash -c 'BASH_ENV=/etc/profile exec bash'"

  # We want to be able to work locally, too
  config.vm.synced_folder "local-repo", "/root/chef-repo"

  # Update apt
  config.vm.provision "shell", inline: <<-SHELL
    echo ">> Updating apt-get..."
    apt-get update
    apt-get install -y git
  SHELL

  # Install RVM as per https://rvm.io/rvm/install
  # (unless it already exists)
  config.vm.provision "shell", inline: <<-SHELL
    echo ">> Installing RVM..."
    gpg --keyserver hkp://keys.gnupg.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3
    [ -x "/usr/local/rvm/bin/rvm" ] || curl -sSL https://get.rvm.io | bash -s stable --ruby
  SHELL

  # Install chef-solo
  # (unless it already exists)
  config.vm.provision "shell", inline: <<-SHELL
    echo ">> Installing chef-client..."
    [ -x "/usr/bin/chef-client" ] || curl -L https://www.opscode.com/chef/install.sh | bash
  SHELL

  # Install chef-librarian
  config.vm.provision "shell", inline: <<-SHELL
    echo ">> Installing librarian-chef...."
    gem install librarian-chef
  SHELL

  #
  # Rather than having it as a separate project, we could sync the folder from local
  # see https://www.vagrantup.com/docs/synced-folders/basic_usage.html
  # But, shared folders means we can't use this same script to deploy to production
  config.vm.provision "shell", inline: <<-SHELL
    echo ">> Cloning openclerk-chef repo..."
    [ -x "/root/chef-repo/.git" ] || git clone https://github.com/soundasleep/openclerk-chef.git /root/chef-repo
    cd /root/chef-repo && git pull
  SHELL

  # Copy over a local web.json here
  # (Yes, we WANT to store our configuration stuff on version control, that's why we're Vagranting)
  # (NOTE that we might need to be smarter with passwords though)
  config.vm.provision "shell", inline: <<-SHELL
    echo ">> Overwriting web.json..."
    cp /vagrant/web.json /root/chef-repo/web.json
  SHELL

  # Now let's get chef to do it's magic for the first time
  # NOTE! It looks like librarian-chef is abandoned, and the new cool technology
  # that all the kids use is called Berkshelf: https://docs.chef.io/berkshelf.html
  # And I think it works the same: https://github.com/englishm/chef-solo-pattern/commits/master
  config.vm.provision "shell", inline: <<-SHELL
    echo ">> Installing cookbooks with librarian-chef..."
    cd /root/chef-repo
    librarian-chef install      # (or `librarian-chef update` to pick up updated cookbooks, very slowly)
  SHELL

  # TODO This is where it falls apart with a '' error.
  config.vm.provision "shell", inline: <<-SHELL
    echo ">> Chef-solo..."
    cd /root/chef-repo
    chef-solo -c solo.rb -j web.json
  SHELL

  config.vm.provision "shell", inline: <<-SHELL
    echo "Hi, world!"
    echo "The current date is $(date) and I'm running on $(hostname) and I am logged in as $USER"
    echo "I am running ruby $(ruby -v) and RVM $(rvm --version)"
    echo "rvm is at $(which rvm) and chef-solo is at $(which chef-solo)"
    ls -la
  SHELL

  # And now if this all works, you should be able to (on your own machine)
  # be able to go to http://localhost:8080 and it will point to http://vagrant:80
end
