# Prerequisites
 
Install ansible: http://docs.ansible.com/ansible/latest/intro_installation.html#installing-the-control-machine

# Add deployment to your project

Execute the following command within your project root directory:

    git submodule add git@github.com:renepardon/ansible-deployment.git .ansible 

# Configuration

1. Copy *environments/shared-vars.yml.example* to *environments/shared-vars.yml* and adjust for your current project
2. Copy *environments/production/hosts.example* to *environments/production/hosts* and set the correct host where you want to deploy the application
3. Copy *roles/deploy/tasks/999-application-specific-tasks.yml.example* to *roles/deploy/tasks/999-application-specific-tasks.yml* and add some deployment specific tasks if required (e.g. clear opcache, create symlinks, run application specific commands)

Why do we need 3.? Because we may have application specific deployment tasks which should be executed. But they differ from application to application so we won't add them to GIT.

# Usage

* Change into *.ansible* directory and execute all commands from within this directory. The .ansible should be a folder within your project. It gets added if you add the project as submodule. This way we can ensure you don't miss any updates 
on the deployment scripts itself

* shared folder usage:


# Prepare/initially setup deployment host

This task has to be run only once to setup the deployment structure on the server side for current application

```bash
ansible-playbook playbook-setup.yml -i environments/production/hosts --ask-become-pass
```

# Prepare deployment

### Available extra vars/CLI parameters

Default deployment branch is master but you can change to a release branch for example if you use git flow or if you want to deploy a specific branch for some reason.

- deploy_branch (master, develop, release/<tagname>) - set to the "release" branch which should be deployed 

### Default command to excute

```bash
ansible-playbook -i localhost, playbook-prepare-deploy.yml --extra-vars "deploy_branch=release/9.9.9"
```

# Deploy

### Available extra vars/CLI parameters

As there are some differences on deployment with Plesk you might want to pass the **server_software** flag so Plesk specific tasks gets triggert during the deployment to sync from current/ symlink to httpdocs/ folder for example.

- server_software (plesk) - false by default
- deploy_type (laravel, symfony) - not defined by default

#### Why do we define CLI parameters instead of configuring them within shared-vars.yml?

We want to use Jenkins in the future and then we have one generic deployment package which can be used for different kind of projects and different kind of server software.

### Full featured example

```bash
ansible-playbook playbook-deploy.yml -i environments/production/hosts --ask-become-pass --extra-vars "deploy_type=laravel server_software=plesk"
```

### Default command to excute

```bash
ansible-playbook playbook-deploy.yml -i environments/production/hosts --ask-become-pass --extra-vars "deploy_type=laravel"
```

The permissions on httpdocs with plesk should be: deployer:psaserv
And all the files below httpdocs: deployer:psacln

/var/www/vhost/application/current -> symlink to latest release <timestamp>
/var/www/vhost/application/releases/<timestamp>/

Where current is the httpdocs folder and contains the files within <timestamp>-folder of the latest release
