# Ansible Dokku Role

This role will install Dokku and requirements with APT package manager.

What will be installed:
- git
- lxc-docker
- daemontools
- build-essential
- libtool
- docker
- dokku

Default Dokku plagins:
- https://github.com/ohardy/dokku-mariadb
- https://github.com/ohardy/dokku-psql