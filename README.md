# AWS VPC Creation with Ansible

This project demonstrates how to create an AWS VPC with the following components using Ansible:

- VPC
- Subnets
- Internet Gateway (IGW)
- Route Table
- Security Group
- EC2 Instances

## Prerequisites

Before you begin, ensure you have the following installed:

- Ansible
- AWS CLI
- AWS credentials configured (via `~/.aws/credentials` or environment variables)

## Project Structure

```
.
├── vpc_create.yml
├── roles
│   └── vpc_creation
│       └── tasks
│           └── main.yml
└── README.md
```

### `create_vpc_and_resources.yml`

```yaml
---
- name: Create VPC and Resources with EC2 Instances
  hosts: localhost
  connection: local
  roles:
    - vpc_creation
```

## Running the Playbook

1. Clone the repository:

```sh
git clone <repository-url>
cd <repository-directory>
```
2. Update the `key_name` in `roles/vpc_creation/tasks/main.yml` to match your AWS key pair.
   
3. Update the access_key and secret_key in yaml file.
  
4. Run the playbook:

```sh
ansible-playbook vpc_create.yml
```
