Ansible-Vagrant-Dokku
=====================

This is an Ansible playbook to setup Dokku environment for production and test

Environment Basic
-----------------
1. Install Brew, Python and pip

2. Install Brew Cask

   ``` bash
      brew install cask
   ```

3. Download and install Ansible

   ``` bash
      pip install ansible
   ```

4. Clone this repo

   ``` bash
      git clone https://github.com/beedesk/deploy.git
   ```

Test Environment
----------------
1. Follow the step of "Environment Basic" (see above)
2. Install vagrant and setup environment

   ``` bash
      # Uses sudo or -K option
      # For verbose output use `-v` option.
      ansible-playbook setup.yml --connection=local -K
   ```
3. Run Vagrant

   ``` bash
      vagrant up
      # Note:
      # Vagrant will set up new VM for you and apply to it `dokku.yml`, that will
      # set up dokku into VM.
      # If something goes wrong you can reapply `dokku.yml` with command:
      # $ vagrant provision
   ```

4. Open [dokku](http://dokku.me/) in browser and finish setup

5. Create and deploy project to test

   ``` bash
      ansible-playbook -i test_hosts project.yml
   ```

You can set variables in `project.yml` playbook or via cli option:

   ```bash
   -e {project_name: python-getting-started, project_url: git@github.com:heroku/python-getting-started.git, project_env: ["key=value", "foo=bar"]}
   ```

Deploy to Production
--------------------
1. Follow the step of "Environment Basic" (see above)
2. Setup production hosts

   ``` bash
      ansible-playbook -i prod_hosts dokku.yml
   ```

3. Connect to your host in browser and finish dokku setup

4. Create and deploy project to prod

   ``` bash
      $ ansible-playbook -i prod_hosts project.yml 
   ```

Background
==========
Here is what each of the steps above does:

setup.yml will:

1. install vagrant and virtualbox (step 1)
2. set ip in Vagrantfile, test_hosts and /etc/hosts (step 3)
3. run vagrant up that triggered dokku.yml
4. add ssh keys (step 3)

dokku.yml will:

1. set swap with roles/swap
2. install dokku + plugins with roles/dokku (apt only install) (step 2, 4)

project.yml will:

1. checkout given project to `apps` directory
2. create database per project (step 5, 6)
3. create and push given project (step 7)

Docs
----
- [dokku application deployment](http://dokku.viewdocs.io/dokku/application-deployment/)
- [heroku python getting started](https://github.com/heroku/python-getting-started)
- [heroku Procfile](https://devcenter.heroku.com/articles/procfile)
- [heroku buildpacks](https://elements.heroku.com/buildpacks)

Steps
-----
1. Download and Install Vagrant
    1. a. Create a new folder << vagrant folder >> (such as `ubuntu-docker`)
    2. `cd << vagrant folder >>`
    3. `vagrant init ubuntu/trusty64`
    4. `vi VagrantFile`
    5. Pick an IP address for Vagrant (such as 10.88.0.8)
    6. change line `config.vm.network "private_network", ip: "<< ip address >>"`
    7. `vagrant up`


2. Install Docker and Dokku
    1. `vagrant ssh`
    2. `apt-get update`
    3. `apt-get install git lxc-docker daemontools build-essential libtool`
    4. Install dokku: (needed to run it twice last time, maybe a transient problem)
        * cd && wget https://raw.github.com/progrium/dokku/v0.3.26/bootstrap.sh && sudo DOKKU_TAG=v0.3.26 bash bootstrap.sh
    5. Increase swap space (http://stackoverflow.com/a/17174672):
        * `sudo dd if=/dev/zero of=/swapfile bs=1M count=1024 && sudo mkswap /swapfile && sudo swapon /swapfile`
    6. Enable swap at boot: add this line to `/etc/fstab`
        * `/swapfile swap swap defaults 0 0`


3. Config host machine
    7. Exit vagrant ssh shell
    8. Edit `/etc/hosts` to add line:
        * << ip address >>   dokku.me
    9. `cat ~/.ssh/id_rsa.pub | vagrant ssh dokku.me "sudo sshcommand acl-add dokku $USER"`
        * (reference: https://github.com/progrium/dokku/blob/master/docs/installation.md)
        * (for AWS test machine, do this: `cat ~/.ssh/id_rsa.pub | ssh -i file.pem.txt ubuntu@<< aws public ip >>.amazonaws.com "sudo sshcommand acl-add dokku $USER"`)


4. Install plugins
    1. log into vagrant `vagrant ssh`
    2. create a sub directory for `plugins`: `mkdir /var/etc/lib/dokku-plugins`
    2. `cd /var/etc/lib/dokku-plugins`
    3. `git clone https://github.com/ohardy/dokku-mariadb.git`
    3. `git clone https://github.com/ohardy/dokku-psql.git`
    6. `cd /var/etc/lib/dokku/plugins`
    3. `ln -s /var/etc/lib/dokku-plugins/dokku-mariadb.git`
    3. `ln -s /var/etc/lib/dokku-plugins/dokku-psql.git`
    8. `sudo dokku plugins:install`

5. Create a MariaDB "project"
    1.


6. Update MySQL

        FLUSH TABLES WITH READ LOCK;
        FLUSH LOGS;
        SET GLOBAL binlog_format = 'MIXED';
        FLUSH LOGS;
        UNLOCK TABLES;

        # http://dba.stackexchange.com/a/6753


7. Create a project and push
    1. export $APP_NAME=<< app name >>
    2. `dokku app:create $APP_NAME`
    3. Add an alias and source it
        * Add this line to ~/.bash_profile: `alias dokkudev='ssh -t dokku@dokku.me'`
        * Source ~/.bash_profile: `source ~/.bash_profile`
    4a. `dokku config:set $APP_NAME KEY_1=VALUE_1 [KEY_2=VALUE_2] ...`
    4b. **Or**, ssh into vagrant `cd << vagrant folder >> && vagrant ssh`
        * And, edit `~dokku/<< app name >>ENV` directly
        * ** remember to include `export` in front of each lines
        * ** do not include any commented line (it will fail subsequence deploy)
        * `dokkudev apps:restart $APP_NAME`  # to cause config reload
    5. `git remote add dokkudev dokku@dokku.me:$APP_NAME`
    6. `git push dokkudev master`
    8. `dokkudev logs $APP_NAME`
    9. If error arise: `setuidgid: fatal: unable to run failure`, do step 2.5 ('daemontools')
