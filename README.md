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

## Role: `vpc_creation`

### `roles/vpc_creation/tasks/main.yml`

```yaml
---
# tasks file for vpc_creation
- name: Create a VPC with dedicated tenancy and a couple of tags
  amazon.aws.ec2_vpc_net:
    name: new_vpc
    cidr_block: 10.0.0.0/16
    access_key: "{{ access_key }}"
    secret_key: "{{ secret_key }}"
    region: us-east-1
    tags:
      module: ec2_vpc_net
      this: works
  register: vpc_id

- name: Create subnet for database servers
  amazon.aws.ec2_vpc_subnet:
    state: present
    access_key: "{{ access_key }}"
    secret_key: "{{ secret_key }}"
    vpc_id: "{{ vpc_id.vpc.id }}"
    cidr: 10.0.1.0/24
    tags:
      Name: Subnet1
  register: subnet_1

- name: Create subnet for web servers
  amazon.aws.ec2_vpc_subnet:
    state: present
    access_key: "{{ access_key }}"
    secret_key: "{{ secret_key }}"
    vpc_id: "{{ vpc_id.vpc.id }}"
    cidr: 10.0.2.0/24
    tags:
      Name: Subnet2
  register: subnet_2

- name: Create Internet Gateway for VPC
  amazon.aws.ec2_vpc_igw:
    aws_access_key: "{{ access_key }}"
    aws_secret_key: "{{ secret_key }}"
    vpc_id: "{{ vpc_id.vpc.id }}"
    region: us-east-1
    state: present
    tags:
      Name: igw
  register: igw

- name: Set up public subnet route table
  amazon.aws.ec2_vpc_route_table:
    vpc_id: "{{ vpc_id.vpc.id }}"
    region: us-east-1
    aws_access_key: "{{ access_key }}"
    aws_secret_key: "{{ secret_key }}"
    tags:
      Name: Public
    subnets:
      - "{{ subnet_1.subnet.id }}"
      - "{{ subnet_2.subnet.id }}"
    routes:
      - dest: 0.0.0.0/0
        gateway_id: "{{ igw.gateway_id }}"
  register: public_route_table

- name: Create Security Group
  amazon.aws.ec2_group:
    aws_access_key: "{{ access_key }}"
    aws_secret_key: "{{ secret_key }}"
    vpc_id: "{{ vpc_id.vpc.id }}"
    region: us-east-1
    state: present
    name: tightsecurity
    description: security_group
    tags:
      Name: tightsecurity
    rules:
    - proto: all
      cidr_ip: 0.0.0.0/0
      rule_desc: allow all traffic
  register: security_group

- name: Create EC2 instance
  amazon.aws.ec2_instance:
    name: "{{ item.name }}"
    key_name: "accesskey"
    vpc_subnet_id: "{{ item.subnet }}"
    instance_type: t2.micro
    security_group: "{{ security_group.group_id }}"
    region: us-east-1
    aws_access_key: "{{ access_key }}"  # From vault as defined
    aws_secret_key: "{{ secret_key }}"  # From vault as defined      
    network:
      assign_public_ip: true
    image_id: "{{ item.image }}"
    tags:
      environment: "{{ item.name }}"
  loop:
    - { image: "ami-04a81a99f5ec58529", name: "ec2_node_1", subnet: "{{ subnet_1.subnet.id }}" }
    - { image: "ami-04a81a99f5ec58529", name: "ec2_node_2", subnet: "{{ subnet_2.subnet.id }}" }

## Running the Playbook

1. Clone the repository:

```sh
git clone <repository-url>
cd <repository-directory>
```

2. Update the `key_name` in `roles/vpc_creation/tasks/main.yml` to match your AWS key pair.

3. Run the playbook:

```sh
ansible-playbook vpc_create.yml
```
