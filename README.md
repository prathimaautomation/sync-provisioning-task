# sync and provisioning task
- configure the vagrantfile to run the provisioning script at the boot-time (at the time of vagrant up)

## create an 'app' folder and create script.sh to include provisioning steps
```script.sh
#!/bin/bash
sudo apt-get update -y
sudo apt-get upgrade -y
sudo apt-get install nginx -y
sudo systemctl status nginx
```
## create a Vagrantfile to include sync app folder and provision to run shell script in VM
```Vagrantfile
Vagrant.configure("2") do |config|
 
# creating a virtual machine ubuntu
 config.vm.box = "ubuntu/xenial64"

# assign private ip to our VM
 config.vm.network "private_network", ip: "192.168.10.100"

# ensure to install hostupdater plugin in our local host before rerunning the vagrant
 config.hostsupdater.aliases = ["development.local"]

# Sync folder from OS to VM
               # "." means current location -  into/inside VM
 config.vm.synced_folder ".", "/home/vagrant/app"          

# running provisioning script with just 'vagrant up' without ssh into VM
 config.vm.provision "shell", path: "./app/script.sh"

end
```
- with vagrant up - it should create the VM - run the script.sh available on our localhost to provision update - upgrade and nginx.
### run 'vagrant up'
```
    $ vagrant up
Bringing machine 'default' up with 'virtualbox' provider...
==> default: Importing base box 'ubuntu/xenial64'...
==> default: Matching MAC address for NAT networking...
    ....
    default:  Main PID: 3542 (nginx)
    default:    CGroup: /system.slice/nginx.service
    default:            ├─3542 nginx: master process /usr/sbin/nginx -g daemon on; master_process on
    default:            ├─3543 nginx: worker process
    default:            └─3544 nginx: worker process
    default:
    default: Nov 08 19:59:23 ubuntu-xenial systemd[1]: Starting A high performance web server and a reverse proxy server...
    default: Nov 08 19:59:23 ubuntu-xenial systemd[1]: Started A high performance web server and a reverse proxy server.
```
#### AIM - should be able to load nginx page without ssh into the VM
![http://192.168.10.100](nginx.png)

[Reference] (https://www.vagrantup.com/docs/provisioning/shell)

#### Creating Dev Test Environment
- copy the 'starter-code' that has actual node app
- create a Vagrantfile to run all the nodeapp dependencies 
```Vagrantfile
Vagrant.configure("2") do |config|

  config.vm.box = "ubuntu/xenial64"
  config.vm.network "private_network", ip: "192.168.10.100"
  config.hostsupdater.aliases = ["development.local"]
  # Sync folder from OS to VM
                       # "." means current location -  into/inside VM
  config.vm.synced_folder ".", "/home/vagrant/app"

end
```
- run the vagrant up to create and run the Virtual Machine
 `vagrant up`
- run the provisioning scripts to install all the nodeapp dependencies
```provision.sh
sudo apt-get update -y
sudo apt-get upgrade -y
sudo apt-get install nginx -y
sudo apt-get install nodejs -y
sudo apt-get install python-software-properties
curl -sL https://deb.nodesource.com/setup_6.x | sudo -E bash –
sudo apt-get install nodejs -y
sudo npm install pm2 -g
```
- we can check if all the dependencies installed as per the requirement can be tested by going into environment/spec-tests folder and by runnning 'rake spec' where all the tests should pass as below:
```
$ rake spec
C:/Ruby26-x64/bin/ruby.exe -I'C:/Ruby26-x64/lib/ruby/gems/2.6.0/gems/rspec-core-3.10.1/lib';'C:/Ruby26-x64/lib/ruby/gems/2.6.0/gems/rspec-support-3.10.3/lib' 'C:/Ruby26-x64/lib/ruby/gems/2.6.0/gems/rspec-core-3.10.1/exe/rspec' --pattern 'spec/default/*_spec.rb'

Package "nginx"
  is expected to be installed

Service "nginx"
  is expected to be running
  is expected to be enabled

Port "80"
  is expected to be listening

Package "nodejs"
  is expected to be installed

Command "nodejs --version"
  stdout
    is expected to match /v6./

Package "pm2"
  is expected to be installed by "npm"

Package "git"
  is expected to be installed

Command "git --version"
  stdout
    is expected to match /2\.7\../

Finished in 3.87 seconds (files took 47.97 seconds to load)
9 examples, 0 failures


Medha@LAPTOP-2CA2UD9P MINGW64 ~/devops_bootcamp/sync_and_provisioning_task/starter-code/environment/spec-tests (main)
$ rake spec
C:/Ruby26-x64/bin/ruby.exe -I'C:/Ruby26-x64/lib/ruby/gems/2.6.0/gems/rspec-core-3.10.1/lib';'C:/Ruby26-x64/lib/ruby/gems/2.6.0/gems/rspec-support-3.10.3/lib' 'C:/Ruby26-x64/lib/ruby/gems/2.6.0/gems/rspec-core-3.10.1/exe/rspec' --pattern 'spec/default/*_spec.rb'

Package "nginx"
  is expected to be installed

Service "nginx"
  is expected to be running
  is expected to be enabled

Port "80"
  is expected to be listening

Package "nodejs"
  is expected to be installed

Command "nodejs --version"
  stdout
    is expected to match /v6./

Package "pm2"
  is expected to be installed by "npm"

Package "git"
  is expected to be installed

Command "git --version"
  stdout
    is expected to match /2\.7\../

Finished in 8.66 seconds (files took 40.94 seconds to load)
9 examples, 0 failures
```
- now can go the nodeapp folder to install npm and start the application as below: 
```
cd app
cd app
npm install
npm start 
```
- Now we can see the app is running on port 3000  
