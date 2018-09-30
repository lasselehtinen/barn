# Barn - Ansible playbooks for Laravel applications
This repository provides Ansible playbook targeted spesifically for Laravel applications. If you are not familiar with Ansible, it is an open-source tool that is most commonly used for provisioning, configuration management and deployment. Together with Ansible and these playbooks you can automate provisioning of your servers and development environment. Please note that Barn is not the designed to handle the actual deployment of the Laravel application. For deployment tools I suggest you checkout [Envoyer](https://envoyer.io/), [Deployer](https://deployer.org) or [Capistrano](http://capistranorb.com/) to name a few. Barn can automatically configures the following:

* PHP
* nginx + php-fpm with multiple virtual hosts
* SSL certificates and renewal with Let's Encrypt
* composer
* redis
* MySQL
* Beanstalk
* elasticsearch
* Crontab entries for queue workers and scheduled tasks
* Laravel Horizon

Barn also tries to apply some additional security by setting SELinux mode to enforcing and enabling automatic package updates. 

## Getting started

### Supported distributions

* CentOS 7
* RedHat (Not tested)
* Ubuntu Trusty
* Debian (Not tested)

Other versions of the distributions listed above might work, but no guarantees given. See testing if you want to try running the distribution of your choice in virtual machines.

### Installing Ansible
You can install Ansible by following the [official installation documentation](http://docs.ansible.com/ansible/intro_installation.html). For Windows 10 users, you can use the instructions for Ubuntu after [installing the Linux subsystem](https://msdn.microsoft.com/en-us/commandline/wsl/install_guide). For older Windows version, please [check this tutorial on running Ansible on cygwin](https://www.jeffgeerling.com/blog/running-ansible-within-windows).

### Clone the repository
```shell
git clone https://github.com/lasselehtinen/barn.git
```

### Configure your inventory files
The inventory files are located in the inventory directory. Check the example for development and read the [Ansible documentation](http://docs.ansible.com/ansible/intro_inventory.html).

### Configure your host variables
Host variables control some of the aspects how the machine is configured. Store the host variable file with the same name as the hostname in the inventory file. Below is an example for the local development machine. 

```yaml
# Let's Encrypt settings
enable_ssl: false
letsencrypt_email: somebody@somewhere.com

# Extra PHP packages
php_extra_packages:    
    - php-intl

# PHP memory limit
php_memory_limit: "512M"

# A list of virtual hosts
virtualhosts:
  blog:
    servernames:
    - blog.development
    - someother.development
    run_queue_worker: true
    run_horizon: false
    has_scheduled_jobs: true
  someothersite:
    servernames:
    - someothersite.development
    
# MySQL root password
mysql_root_password: somesecret

# List of MySQL databases
mysql_databases:
  blog:
     mysql_user: blog
     mysql_password: table_password
  someothersite:
     mysql_user: otherdbuser
     mysql_password: table_password
```

### Create .env templates
Create .env file templates in the dotenv_templates folder with the name `{{hostname}}.{{virtualhost}}.j2`. For example host centos7-test has a virtualhost called blog so the template should be named `centos7-test.blog.j2`. This is completely optional.

### Securing your host variables
Since you storing highly confidential information like production database passwords in the host variables and .env templates it is highly recommend that encrypt them with a symmetric AES key. Luckily Ansible has built-in tool for this called Vault. Please [read the Vault documentation](http://docs.ansible.com/ansible/playbooks_vault.html) on how to set it up.

#### Supported variables

| Name                                   | Description                                                                                                            | Required |
|----------------------------------------|------------------------------------------------------------------------------------------------------------------------|----------|
| enable_ssl                             | Controls whether the Playbook configures Let's Encrypt certificates on all the virtual hosts.                          | No       |
| letsencrypt_email                      | Email address for sending the expiry notices for the certificates.                                                     | No       |
| php_extra_packages                     | List of extra php-packages you want to install                                                                         | No       |
| php_memory_limit                       | Sets the memory_limit in php.ini                                                                                       | No       |
| virtualhosts.name                      | Shortname used for folders like /var/www/name/public                                                                   | Yes      |
| virtualhosts.name.servernames          | List of hostnames for the virtual host. Must be a valid FQDN if you set enable_ssl to true.                            | Yes      |
| virtualhosts.name.run_queue_worker     | Sets whether we should run artisan queue:work on this virtualhost                                                      | No       |
| virtualhosts.name.run_horizon          |  Sets whether Horizon should be running on this virtual host. Do not set both run_queue_worker and run_horizon to true | No       |
| virtualhosts.name.queue_worker_timeout | Sets the timeout for the queue worker, default is 60 seconds                                                           | No       |
| virtualhosts.name.has_scheduled_jobs   | Sets whether we should run artisan schedule:run every minute on this virtualhost                                       | No       |
| mysql_root_password                    | Root password for MySQL                                                                                                | Yes      |
| mysql_databases.name                   | Name of MySQL database that will generated                                                                             | Yes      |
| mysql_databases.name.mysql_user        | Name of normal MySQL user for that database. Place this in your .env file.                                             | Yes      |
| mysql_databases.name.mysql_password    | Password for the MySQL user. Place this in your .env file.                                                             | Yes      |                                   | Yes       |     |

## Running Barn

### Checking SSH access
After you have done with the configuration make sure you have SSH access to the servers that listed on the inventory file. You can test this by using the ping module. Troubleshooting the SSH connection is the out of scope of this repository. Please note that you can set the SSH user and port in the inventory file, see the development for example.

```shell
ansible -i inventory/development webservers -m ping
localhost | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

### Running the Playbook
You can run the playbook on all hosts with the following command:
```shell
ansible-playbook -i inventory site.yml
```
Provisioning takes a while for the first time so go grab a coffee. If there were no problems, you should have readily provisioned machine and you can deploy your Laravel application to the /var/www/`virtualhosts.name` folder.  

### Running custom playbooks
Sometimes you have something specific which Barn does not cover. In that case you can create your custom playbooks. Store the playbooks to `roles/custom/files` with the .yml extension and they will be executed on all roles.

## Contributing

Pull requests are welcome. Repository provides a Vagrant file that spins up virtual machines on the supported distributions. Please make sure that running the command below completes without any failures. If you want to add support for a new distribution, add a new base box to the Vagrant file and add the host to the inventory/testing file. Remember also to copy the existing host_vars file for the new distribution. 

```shell
ansible-playbook -i inventory/testing site.yml
```

## Issues

If you have problems or suggestions, please [open a new issue in GitHub](https://github.com/lasselehtinen/barn/issues). 

## License

The MIT License (MIT). Please see [License File](LICENSE.md) for more information.
