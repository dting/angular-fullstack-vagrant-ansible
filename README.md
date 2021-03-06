# angular-fullstack-vagrant-ansible

A Vagrantfile provisioned with Ansible that sets up a dev environment for working on Angular-Fullstack projects.

This is only usable on OSX at the moment because the box uses NFS to mount the share folder.

It could be modified easily to be used with a windows machine by removing a couple lines or adding smb support. See https://github.com/blinkreaction/boot2docker-vagrant for a reference to how that might be accomplished.

Follow up to my attempt at using docker for setting up a dev environment for OSX for completing [**FreeCodeCamp**](http://www.freecodecamp.com/) [basejumps](http://www.freecodecamp.com/challenges/waypoint-get-set-for-basejumps):

https://github.com/dting/fcc-angular-fullstack-docker

## Example Project

https://github.com/dting/ng-fs-vagrant-ansible

## Prerequisites

1. Vagrant (http://docs.vagrantup.com/v2/installation/index.html)
2. Virtualbox (https://www.virtualbox.org/wiki/Downloads)
3. Ansible (http://docs.ansible.com/ansible/intro_installation.html#installation)

If you have homebrew (http://brew.sh/) and homebrew cask (http://caskroom.io/):

    $ brew cask install virtualbox
    $ brew cask install virtualbox-extension-pack
    $ brew cask install vagrant
    $ brew install ansible

## Setup and Usage

Clone repository into a folder that will become your app's vagrant folder and change to that directory.

    $ git clone git@github.com:dting/angular-fullstack-vagrant-ansible.git myProject
    $ cd myProject

**_[ Danger! Make sure to be inside the new project directory before continuing! ]_**

Remove git files before you start.

    myProject$ find . | grep .git | xargs rm -rf

If you would like the provisioner to setup your git global config with your username and email, in `provisioning/main.yml` edit:

    - { role: web, git_username: someone, git_email: someone@example.com }

Bring up the vm and wait for ansible to provision it.

    myProject$ vagrant up

This first vagrant up will take a fairly long time to complete (~30 mins).
When this completes ssh into the vm. When working on the project you should probably leave a terminal window up connected to this vm. If you need to install bower components or npm packages it should be done in this vm or by editing the provisioning files.

    myProject$ vagrant ssh

You should see some banner text and a prompt:

    vagrant@dev:/vagrant/app$

Run the `yo angular-fullstack generator` and choose the options you want to use:

    vagrant@dev:/vagrant/app$ yo angular-fullstack

The generator will install the bower and npm packages needed for the project.

You should notice that these files are being created on your host machine. The `myProject` folder is mounted at `/vagrant` on this vm. The generator was run in the `/vagrant/app` folder meaning that now on the host machine the files are in the `myProject/app` folder.

At this point running `grunt serve` should bring up the server.

    vagrant@dev:/vagrant/app$ grunt serve

From the host computer we can use the browser and see the page at `192.168.111.222:9000`. Any changes made to the files in the app should trigger a reload and we should see the changes reflected immediately. Instead of killing that task we can bring up another terminal and ssh into the same vm.

    myProject$ vagrant ssh
    vagrant@dev:/vagrant/app$

Let's install some npm packages and login to heroku.

    vagrant@dev:/vagrant/app$ npm install grunt-contrib-imagemin --save-dev && npm install --save-dev && heroku login

Go back to the original terminal with the running `grunt serve` process and kill it with `ctrl-c`.  
We are going to deploy to heroku using the `angular-fullstack` heroku deployment generator.  
Before we do this, we probably want to set the git config global user.name and user.email if you did not modify the variables in the `provisioning/main.yml` provisioning file.

    vagrant@dev:/vagrant/app$ git config --global user.email "me@example.com"
    vagrant@dev:/vagrant/app$ git config --global user.name "Me"
    vagrant@dev:/vagrant/app$ yo angular-fullstack:heroku

Choose a name and a region and the app should be deployed.
After that completes we need to add the heroku mongolab addon and set the env var:

    vagrant@dev:/vagrant/app$ cd dist
    vagrant@dev:/vagrant/app$ heroku config:set NODE_ENV=production && heroku addons:add mongolab

We should now be able to view the app deployed the heroku. For our example it is at [`https://ng-fs-vagrant-ansible.herokuapp.com/`](https://ng-fs-vagrant-ansible.herokuapp.com/).

## Git

At some point you will want to create a git repository with your code. It is up to you to decide if you want to include the Vagrantfile and provisioning directory along with your app.

    myProject$ git init
    myProject$ git commit -m "Initial commit."

or

    myProject/app$ git init
    myProject/app$ git commit -m "Initial commit."

If ssh-agent-forwarding is setup git can also be used inside the vm (see more about this later).

## Workflow

The editing of the files should be done on the host machine. The changes will be picked up by the vm and reflected via livereload to the host browsers pointed at the `http://<vm's ip (default:192.168.111.222)>:9000` when the `grunt serve` task is running in the vm.

## ssh-agent-forwarding

ssh-agent-forwarding is used so git can be used seemlessly on the vagrant web vm. You can read an in depth over article about it here:

http://www.unixwiz.net/techtips/ssh-agent-forwarding.html

On the host in `~/.ssh/config` add:

    Host 192.168.111.222
      ForwardAgent yes

Store the pass phrases on your keychain:

    $ ssh-add -K

From inside the web vm:

    vagrant@dev:/vagrant/app$ ssh -T git@github.com

and you should see something like:

    The authenticity of host 'github.com (192.30.252.131)' can't be established.
    RSA key fingerprint is 16:27:ac:a5:76:28:2d:36:63:1b:56:4d:eb:df:a6:48.
    Are you sure you want to continue connecting (yes/no)? yes
    Warning: Permanently added 'github.com,192.30.252.131' (RSA) to the list of known hosts.
    Hi dting! You've successfully authenticated, but GitHub does not provide shell access.
