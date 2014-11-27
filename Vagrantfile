# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  
  # All Vagrant configuration is done here. The most common configuration
  # options are documented and commented below. For a complete reference,
  # please see the online documentation at vagrantup.com.
  
  #     _        _     __  __                                             
  #    / \   ___| |_  |  \/  | ___  _ __   __ _ _   _  ___ _   _ _ __ ___ 
  #   / _ \ / __| __| | |\/| |/ _ \| '_ \ / _` | | | |/ _ \ | | | '__/ __|
  #  / ___ \ (__| |_  | |  | | (_) | | | | (_| | |_| |  __/ |_| | |  \__ \
  # /_/   \_\___|\__| |_|  |_|\___/|_| |_|\__, |\__,_|\___|\__,_|_|  |___/
  #                                       |___/                           
  #                                                                       
  
  config.vm.define "voyager", primary: true do |voyager|
  
  # The hostname the machine should have.
  # Defaults to nil. If nil, Vagrant won't manage the hostname.
  # If set to a string, the hostname will be set on boot.
  voyager.vm.hostname = "voyager"
  
  # This sets the username that Vagrant will SSH as by default.
  # Providers are free to override this if they detect a more appropriate user.
  # By default this is "vagrant," since that is what most public boxes are made
  # as.
voyager.ssh.username = "act_developer"
  
################################################################################
#
# ACT-VOYAGER INSTALL SCRIPT   BEGIN
#
################################################################################

$provision_script = <<'END_OF_PROVISION_SCRIPT'
#/ bin/bash


clear
cat <<'BANNER'
    _        _    __     __                                
   / \   ___| |_  \ \   / /__  _   _  __ _  __ _  ___ _ __ 
  / _ \ / __| __|  \ \ / / _ \| | | |/ _` |/ _` |/ _ \ '__|
 / ___ \ (__| |_    \ V / (_) | |_| | (_| | (_| |  __/ |   
/_/   \_\___|\__|    \_/ \___/ \__, |\__,_|\__, |\___|_|   
                               |___/       |___/           

BANNER
sleep 3

#
# run this as local user
#

sudo -u act_developer -i
cd /home/act_developer

export ACT_USER="/home/act_developer"
export ACT_HOME="$ACT_USER/Act"
export PERL5LIB="$ACT_HOME/lib"
export ACTHOME=$ACT_HOME

export ACT_CONF="voyager"

#
# install the Act software from github...
#

git clone https://github.com/Act-Voyager/Act.git $ACT_HOME

#
# cpanm is smart enough to handle the whole distribution at once
#
# just make sure that Module::Install has been installed
# just make sure that there is a valid Act config
#

cpanm --sudo --installdeps $ACT_HOME

#
# Case Sensitive workaround
#

ln -s Act act

#
# create dir
#

mkdir -p $ACT_HOME/wwwdocs/$ACT_CONF
mv $ACT_HOME/skel/wwwdocs/*   $ACT_HOME/wwwdocs/$ACT_CONF

mkdir -p $ACT_HOME/actdocs/$ACT_CONF
mv $ACT_HOME/eg/conf          $ACT_HOME/
mv $ACT_HOME/skel/actdocs/*   $ACT_HOME/actdocs/$ACT_CONF

mkdir -p $ACT_HOME/var
mkdir -p $ACT_HOME/conf/apache

#
# $ACT_HOME/conf/act.ini
#

cat >$ACT_HOME/conf/act.ini <<'EOF'
[general]
conferences = test
cookie_name = act
searchlimit = 20
dir_photos  = photos
dir_ttc     = /home/act_developer/Act/var
max_imgsize = 320x200

[database]
name        = act
dsn         = dbi:Pg:dbname=act_sample
user        = actuser_data
passwd      = xyzzy;

test_dsn    = dbi:Pg:dbname=acttest
test_user   = actuser_data
test_passwd = xyzzy;

[email]
sendmail    = /usr/sbin/sendmail
test        = 0
sender_address = act_tester@mongueurs.local

[wiki]
dbname      = act_sample_wiki
dbuser      = actuser_wiki
dbpass      = xyzzy;

[payment]
open      = 0
invoices  = 0
type        = Fake
notify_bcc  = payments@mongueurs.local

[payment_type_Fake]
plugin = Fake

[flickr]
# see http://www.flickr.com/services/api/
apikey  = 0123456789ABCDEF0123456789ABCDEF
EOF

#
# $ACT_HOME/conf/local.ini
#

cat >$ACT_HOME/conf/local.ini <<'EOF'
[general]
default_language = en
languages = en
name_en = Perl Event Name
default_country = fr
full_uri = http://localhost:8080/
timezone = Europe/Paris

[talks]
durations = 20 40 120
start_date = 2014-08-06 18:00:00
end_date = 2014-08-07 18:00:00
submissions_open = 0
show_schedule = 0

[rooms]
rooms = roomA roomB
roomA_name_en = Room A
roomB_name_en = Room B

[database]
dump_file = act.dump
pg_dump = /usr/bin/pg_dump

[payment]
currency = EUR
type_fake_notify_bcc = paymentbcc@localhost
products = registration

[product_registration]
prices = 1
name_en = Registration
[product_registration_price1]
amount = 25
EOF

#
# add VirtualHost to httpd.conf
#

echo "Include $ACT_HOME/conf/apache" >>/usr/local/apache/conf/httpd.conf"

sudo sed -i 's/User nobody/User act_developer/g'    /usr/local/apache/conf/httpd.conf
sudo sed -i 's/Group nogroup/Group act_developer/g' /usr/local/apache/conf/httpd.conf

cat >$ACT_HOME/conf/apache/act_main.conf <<"EOF"
PerlSetupEnv On
PerlPassEnv ACTHOME
# mod_perl initialisation
PerlRequire $ACTHOME/conf/startup.pl
EOF

cat >$ACT_HOME/conf/apache/conf_$ACT_CONF.conf <<'EOF'
Listen 8080
<VirtualHost *:8080>
      ServerName   localhost:8080
      ServerAdmin  webmaster@example.com
      DocumentRoot /home/act_developer/Act/wwwdocs
      Include      /home/act_developer/Act/conf/httpd.conf
</VirtualHost>
EOF

#
# restart Apache httpd
#

sudo ACTHOME=$ACT_HOME PERL5LIB=$PERL5LIB /usr/local/apache/bin/apachectl graceful

echo "running cpanm... please wait"
# we silently let this run and FAIL and then do it again, which succeeds
cpanm --sudo $ACT_HOME >/dev/null 2>/dev/null
echo "that usually fail... 1/2605 tests" 1>&2
echo
sleep 3
echo "lets do it again...."
cpanm --sudo $ACT_HOME
#
# that should do it

# DONE!!!!

END_OF_PROVISION_SCRIPT

################################################################################
#
# ACT-VOYAGER INSTALL SCRIPT   END
#
################################################################################

#voyager.vm.provision "shell", path: "bin/github-clone-and-make.sh"
voyager.vm.provision "shell", inline: $provision_script


  # The path to the private key to use to SSH into the guest machine.
  # By default this is the insecure private key that ships with Vagrant, since
  # that is what public boxes use. If you make your own custom box with a custom
  # SSH key, this should point to that private key.
  # You can also specify multiple private keys by setting this to be an array.
  # This is useful, for example, if you use the default private key to bootstrap
  # the machine, but replace it with perhaps a more secure key later.
  config.ssh.private_key_path = [ '~/.vagrant.d/insecure_private_key' ]
  
  end # vm.define. "voyager"
  
  # Every Vagrant virtual environment requires a box to build off of.
  config.vm.box     = "voyager"
  config.vm.box_url = "http://thema-media.nl/act-out-of-the-box/package.box"
# config.vm.box_url = "file:///Users/hoesel_v_tj/Documents/ACT%20Voyager/Vagrant/package.box"
  
  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false
  
  # apache
  config.vm.network "forwarded_port", guest: 8080, host: 8080
  
  # apt-cacher-ng running on the host as an apt proxy
  #config.vm.network "private_network", ip: "192.168.42.42"
  config.vm.network :private_network, type: :dhcp
  
  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  config.vm.network "public_network"
  
  # If true, then any SSH connections made will enable agent forwarding.
  # Default value: false
  # config.ssh.forward_agent = true
  
  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  config.vm.synced_folder "./Act", "/home/act_developer/Act", create: true
  
  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  # config.vm.provider "virtualbox" do |vb|
  #   # Don't boot with headless mode
  #   vb.gui = true
  #
  #   # Use VBoxManage to customize the VM. For example to change memory:
  #   vb.customize ["modifyvm", :id, "--memory", "1024"]
  # end
  #
  # View the documentation for the provider you're using for more
  # information on available options.
  
  # Enable provisioning with CFEngine. CFEngine Community packages are
  # automatically installed. For example, configure the host as a
  # policy server and optionally a policy file to run:
  #
  # config.vm.provision "cfengine" do |cf|
  #   cf.am_policy_hub = true
  #   # cf.run_file = "motd.cf"
  # end
  #
  # You can also configure and bootstrap a client to an existing
  # policy server:
  #
  # config.vm.provision "cfengine" do |cf|
  #   cf.policy_server_address = "10.0.2.15"
  # end
  
  # Enable provisioning with Puppet stand alone.  Puppet manifests
  # are contained in a directory path relative to this Vagrantfile.
  # You will need to create the manifests directory and a manifest in
  # the file default.pp in the manifests_path directory.
  #
  # config.vm.provision "puppet" do |puppet|
  #   puppet.manifests_path = "puppet/manifests"
  #   puppet.module_path    = "puppet/modules"
  #   puppet.manifest_file  = "site.pp"
  #   puppet.options        = ["--verbose","--show_diff"]
  # end
  
  # Enable provisioning with chef solo, specifying a cookbooks path, roles
  # path, and data_bags path (all relative to this Vagrantfile), and adding
  # some recipes and/or roles.
  #
  # config.vm.provision "chef_solo" do |chef|
  #   chef.cookbooks_path = "../my-recipes/cookbooks"
  #   chef.roles_path = "../my-recipes/roles"
  #   chef.data_bags_path = "../my-recipes/data_bags"
  #   chef.add_recipe "mysql"
  #   chef.add_role "web"
  #
  #   # You may also specify custom JSON attributes:
  #   chef.json = { mysql_password: "foo" }
  # end
  
  # Enable provisioning with chef server, specifying the chef server URL,
  # and the path to the validation key (relative to this Vagrantfile).
  #
  # The Opscode Platform uses HTTPS. Substitute your organization for
  # ORGNAME in the URL and validation key.
  #
  # If you have your own Chef Server, use the appropriate URL, which may be
  # HTTP instead of HTTPS depending on your configuration. Also change the
  # validation key to validation.pem.
  #
  # config.vm.provision "chef_client" do |chef|
  #   chef.chef_server_url = "https://api.opscode.com/organizations/ORGNAME"
  #   chef.validation_key_path = "ORGNAME-validator.pem"
  # end
  #
  # If you're using the Opscode platform, your validator client is
  # ORGNAME-validator, replacing ORGNAME with your organization name.
  #
  # If you have your own Chef Server, the default validation client name is
  # chef-validator, unless you changed the configuration.
  #
  #   chef.validation_client_name = "ORGNAME-validator"

end # Vagrant.configure
