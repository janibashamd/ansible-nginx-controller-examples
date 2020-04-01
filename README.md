NGINX Controller Example
=========

Requirements
---------------

- Ansible 2.7+
- [Python boto3 library](https://aws.amazon.com/sdk-for-python/)
- [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-install.html)
- A default Access Key [configured in aws cli](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html)


The whats
------------

note: all playbooks tested with Ubuntu 18.04 minimum AMI image

`demo-vpc.json` | Cloud Formation template to create a VPC with 2 public and 2 private subnets.

`ansible-playbook controller_environment_deploy_aws.yaml`

- Deploys the machines to the vpc
- uses [controller_aws_vars.yaml](controller_aws_vars.yaml)
- writes individual inventory files in `playbook_dir`

`ansible-playbook nginx_controller_postgresql_install.yaml -i dbserver`

- installs PostgreSQL, naas user, naas database
- for some reason unclear to me at this time, this needs to be run a second time to overcome a failure

`ansible-playbook nginx_install_controller.yaml -i controller`

- installs Controller
- reads dbserver inventory file
- sets root password for su task (Ubuntu 18.04)
- requires [nginx_install_controller_vars.yaml](nginx_install_controller_vars.yaml)

`nginx_installer_controller_vars.yaml` | your Controller settings
`nginx_install_controller.yaml` | installs nginx controller and pre-requisites

`ansible-playbook nginx_controller_license.yaml -e "nginx_install_controller_vars.yaml"`
- installs the Controller license

`nginx_controller_agent_2.x.yaml` | installs the nginx Controller agent on an nginx plus instance (see [my nginx samples] (<https://github.com/brianehlert/ansible-nginx-examples)> )
`ansible-playbook nginx_controller_agent_2.x.yaml -i loadbalancers`

- for NGINX Controller 2.x family
- reads controller inventory file
- requires controller_user_email, controller_password

`nginx_controller_agent_3.x.yaml` | same as the 2.x version but handles the difference in Controller authentication return
`ansible-playbook nginx_controller_agent_3.x.yaml -i loadbalancers`

- for NGINX Controller 3.x family
- reads controller inventory file
- requires controller_user_email, controller_password

`nginx-plus-api.conf` and `stub_status.conf` | these fully enable all nginx metrics in Controller for the instance

`docker-daemon.json` | this sets the Docker daemon behavior according to Docker recommendations (cgroup driver) and sets up log rotation to prevent filling the partition, and the storage driver for Kubernetes.  So everyone is happy.
