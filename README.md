# Barn - Ansible playbooks for Laravel applications
This repository provides Ansible playbook targeted spesifically for Laravel applications. If you are not familiar with Ansible, it is an open-source tool that is most commonly used for provisioning, configuration management and deployment. Together with Ansible and these playbooks you can automate provisioning of your servers and development environment. Please note that Barn is not the designed to handle the actual deployment of the Laravel application. For deployment tools I suggest you checkout [Envoyer](https://envoyer.io/), [Deployer](https://deployer.org) or [Capistrano](http://capistranorb.com/) to name a few. Barn can automatically configures the following:

* PHP
* nginx + php-fpm with multiple virtual hosts
* SSL certificates and renewal with Let's Encrypt
* redis
* MySQL
* Beanstalk
* Crontab entries for queue workers and scheduled tasks

Barn also tries to apply some additional security by setting SELinux mode to enforcing and enabling automatic package updates. Currenly only CentOS is supported. 

## Getting started

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

# A list of virtual hosts
virtualhosts:
  blog:
    servername: blog.development
    run_queue_worker: true
    has_scheduled_jobs: true
  someothersite:
    servername: someothersite.development
    
# MySQL root password
mysql_root_password: somesecret

# List of MySQL databases
mysql_databases:
  blog:
     mysql_user: blog
     mysql_password: table_password
  someothersite:
     mysql_user: someothersitedbuser
     mysql_password: table_password
```

### Securing your host variables
Since you storing highly confidential information like production database passwords in the host variables file it is highly recommend that encrypt them with a symmetric AES key. Luckily Ansible has built-in tool for this called Vault. Please [read the Vault documentation](http://docs.ansible.com/ansible/playbooks_vault.html) on how to set it up.

#### Supported variables

| Name                                  | Description                                                                                     | Required  |
|-------------------------------------- |-----------------------------------------------------------------------------------------------  |---------- |
| enable_ssl                            | Controls whether the Playbook configures Let's Encrypt certificates on all the virtual hosts.   | No        |
| letsencrypt_email                     | Email address for sending the expiry notices for the certificates.                              | No        |
| virtualhosts.name                     | Shortname used for folders like /var/www/name/public                                            | Yes       |
| virtualhosts.name.servername          | Hostname for the virtual host. Must be a valid FQDN if you set enable_ssl to true.              | Yes       |
| virtualhosts.name.run_queue_worker    | Sets whether we should run artisan queue:work on this virtualhost                               | No        |
| virtualhosts.name.has_scheduled_jobs  | Sets whether we should run artisan schedule:run every minute on this virtualhost                | No        |
| mysql_root_password                   | Root password for MySQL                                                                         | Yes       |
| mysql_databases.name                  | Name of MySQL database that will generated                                                      | Yes       |
| mysql_databases.name.mysql_user       | Name of normal MySQL user for that database. Place this in your .env file.                      | Yes       |
| mysql_databases.name.mysql_password   | Password for the MySQL user. Place this in your .env file.                                      | Yes       |     |

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

## Issues

If you have problems or suggestions, please [open a new issue in GitHub](https://github.com/lasselehtinen/barn/issues). 

## License

The MIT License (MIT). Please see [License File](LICENSE) for more information.
