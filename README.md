# ormuco-deploy

## Prerequisites

This project has the following required system packages

- python-pip
- virtualenv

## Install dependencies

To install dependencies, create a Python virtualenv and use Pip:

```bash
virtualenv venv
source venv/bin/activate
pip install -r requirements.txt
```

For next part, we assume that this virtualenv has been created and is still activated.

## Deploy ormuco-app to EC2 instance

### Inventory

EC2 instances are specified in **inventory** file:

```ini
[all]
ormuco.davidcharbonnier.fr
```

We assume that SSH connection to hosts is done with **ec2-user** username and that SSH private key has been added to ssh-agent:

```bash
ssh-add <keyfile.pem>
```

### Variables

Variables are set both in **site.yml** file containing playbook and in role's defaults.

In playbook are defined following vars:

- ansible_python_interpreter: which Python version use (Python 3 by default but some modules require Python 2 interpreter)
- ormuco_fresh_install: will this Ansible playbook run should perform a fresh install (ie. wipe and init database)?
- ormuco_application_replicas: number of application replicas
- ormuco_environment: Flask application environment
- ormuco_secret_key: Flask application secret key (vaulted value)
- ormuco_database_host: database host
- ormuco_database_port: database port
- ormuco_database_name: database name (vaulted value)
- ormuco_database_username: database username (vaulted value)
- ormuco_database_password: database password (vaulted value)

In ormuco role's default are defined following vars:

- ormuco_system_packages: list of prerequisites system packages
- ormuco_application_path: where to install application
- ormuco_application_requirements_file_path: where to find Python requirements file
- ormuco_environment: default Flask application environment (overriden by playbook value)

All of these values can be overriden on command line with `-e` switch.

### Lauch playbook

In order to perform a fresh install, launch following command:

```bash
ansible-playbook -i inventory site.yml --ask-vault-pass -e ormuco_fresh_install=true
```

To update application to latest revision, launch the command (with **ormuco_fresh_install** value not updated in playbook):

```bash
ansible-playbook -i inventory site.yml --ask-vault-pass
```

Or:

```bash
ansible-playbook -i inventory site.yml --ask-vault-pass -e ormuco_fresh_install=false
```

To update and scale application to a new number of replicas, 10 here:

```bash
ansible-playbook -i inventory site.yml --ask-vault-pass -e ormuco_application_replicas=10
```

Be aware that after scaling application, you need to report this value in playbook for future run, if not, application will be scaled down to playbook replicas value.

## Not yet implemented

Here is list of improvement that could be implemented:

- Enable SELinux with enforcing mode
- Reinforce Docker socket security by using TLS authentication
- Implement sticky session on Traefik reverse proxy in case ormuco application starts using sessions and become stateful (or use a shared storage to store sessions)
- Implement autoscaling mechanism based on Traefik metrics API (Prometheus scraping endpoint)
