## Automated ELK Stack Deployment

The files in this repository were used to configure the network depicted below.

![Network Topology](https://raw.githubusercontent.com/Zyoung28/ELK-Stack-Project/main/Diagrams/Network%20Topology.png)

These files have been tested and used to generate a live ELK deployment on Azure. They can be used to either recreate the entire deployment pictured above. Alternatively, select portions of the YAML file may be used to install only certain pieces of it, such as Filebeat.

  /ansible/filebeat-playbook.yml
  
  ---
  - name: installing and launching filebeat
    hosts: webservers
    become: yes
    tasks:

    - name: download filebeat deb
      command: curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.6.1-amd64.deb

    - name: install filebeat deb
      command: dpkg -i filebeat-7.6.1-amd64.deb

    - name: drop in filebeat.yml
      copy:
        src: /etc/ansible/files/filebeat-config.yml
        dest: /etc/filebeat/filebeat.yml

    - name: enable and configure system module
      command: filebeat modules enable system

    - name: setup filebeat
      command: filebeat setup

    - name: start filebeat service
      command: service filebeat start

    - name: enable service filebeat on boot
      systemd:
        name: filebeat
        enabled: yes
        masked: no

This document contains the following details:
- Description of the Topologu
- Access Policies
- ELK Configuration
  - Beats in Use
  - Machines Being Monitored
- How to Use the Ansible Build


### Description of the Topology

The main purpose of this network is to expose a load-balanced and monitored instance of DVWA, the D*mn Vulnerable Web Application.

Load balancing ensures that the application will be highly efficient, in addition to restricting access to the network.
- _TODO: What aspect of security do load balancers protect? What is the advantage of a jump box?_
- 
Integrating a ELK server enables users with tools to monitor vulnerable VMs for changes to filesystems and system performance. The ELK stack is separated by region and virtual network. This helps separate operational services with production services

Integrating an ELK server allows users to easily monitor the vulnerable VMs for changes to the _____ and system _____.
- _TODO: What does Filebeat watch for?_ Lightweight and easy to use shipper for forwarding and centralizing log dat
- _TODO: What does Metricbeat record?_ Periodically collects metrics from operating system and other running services

The configuration details of each machine may be found below.
_Note: Use the [Markdown Table Generator](http://www.tablesgenerator.com/markdown_tables) to add/remove values from the table_.

| Name     | Function | IP Address | Operating System |
|----------|----------|------------|------------------|
| Jump box | Gateway  | 10.0.0.4   | Linux            |
| Web-1    | server   | 10.0.0.5   | Linux            |
| Web-2    | server   | 10.0.0.6   | Linux            |
| ELK VM   | Server   | 10.2.0.4   | Linux            |

### Access Policies

The machines on the internal network are not exposed to the public Internet. 

Only the _____ machine can accept connections from the Internet. Access to this machine is only allowed from the following IP addresses:
- _TODO: Add whitelisted IP addresses_ My Home IP address: 76.229.189.236
Machines within the network can only be accessed by the Jump Box or the Elk VM.
Jump Box IP: 10.0.0.4 Elk VM IP: 10.2.0.4

A summary of the access policies in place can be found in the table below.

| Name     | Publicly Accessible | Allowed IP Addresses |
|----------|---------------------|----------------------|
| Jump Box | Yes                 |  76.229.189.236      |
| Web-1    | No                  |  Vnet access only    |
| Web-2    | No                  |  Vnet access only    |
| ELK VM   | Yes                 |  76.229.189.236      |

### Elk Configuration

Ansible was used to automate configuration of the ELK machine. No configuration was performed manually, which is advantageous because...
- _TODO: What is the main advantage of automating configuration with Ansible?_
Multiple servers can be managed through a single yml script allowing for quick deployment of needed tools such as metricbeat or filebeat.

The playbook implements the following tasks:

Below is the ELK installation script. Docker and Python are installed. More memory needs to be used for the VM to run correctly. The ELK container along with the ports ELK uses need to be installed / configured. The Hosts file needs to have the elk server added seperatley from any other servers you do not want ELK installed on with the ip listed.

---
  - name: Configure Elk VM with Docker
    hosts: elk
    remote_user: azadmin
    become: true
    tasks:
    # Use apt module
    - name: Install docker.io
      apt:
        update_cache: yes
        force_apt_get: yes
        name: docker.io
        state: present

      # Use apt module
    - name: Install python3-pip
      apt:
        force_apt_get: yes
        name: python3-pip
        state: present

      # Use pip module (It will default to pip3)
    - name: Install Docker module
      pip:
        name: docker
        state: present

      # Use command module
    - name: Increase virtual memory
      command: sysctl -w vm.max_map_count=262144

      # Use sysctl module
    - name: Use more memory
      sysctl:
        name: vm.max_map_count
        value: "262144"
        state: present
        reload: yes

      # Use docker_container module
    - name: download and launch a docker elk container
      docker_container:
        name: elk
        image: sebp/elk:761
        state: started
        restart_policy: always
        # Please list the ports that ELK runs on
        published_ports:
          - 5601:5601
          - 9200:9200
          - 5044:5044

      # Use systemd module
    - name: Enable service docker on boot
      systemd:
        name: docker
        enabled: yes


The following screenshot displays the result of running `docker ps` after successfully configuring the ELK instance.

![TODO: Update the path with the name of your screenshot of docker ps output](Images/docker_ps_output.png)

### Target Machines & Beats
This ELK server is configured to monitor the following machines:
- _TODO: List the IP addresses of the machines you are monitoring_
Web-1: 10.0.0.5
Web-2: 10.0.0.6
We have installed the following Beats on these machines:
- _TODO: Specify which Beats you successfully installed_
- Filebeat
- Metricbeat

These Beats allow us to collect the following information from each machine:
- _TODO: In 1-2 sentences, explain what kind of data each beat collects, and provide 1 example of what you expect to see. E.g., `Winlogbeat` collects Windows logs, which we use to track user logon events, etc._
1. Filebeat collects Operating System log information allowing the tracking of user logon events, failed access attempts, service starts and stops.
2. Metricbeat collects system metrics from the web servers. For example cpu usage, memory usage, drive space usage and drive space available for each host.

### Using the Playbook
In order to use the playbook, you will need to have an Ansible control node already configured. Assuming you have such a control node provisioned: 

SSH into the control node and follow the steps below:
- Copy the ansible.cfg file to /etc/ansible.
- Update the hosts file to include...
- Run the playbook, and navigate to ansible/yml to check that the installation worked as expected.

_TODO: Answer the following questions to fill in the blanks:_
- _Which file is the playbook? Where do you copy it?_ /etc/ansible/file/filebeat-config.yml
- _Which file do you update to make Ansible run the playbook on a specific machine? How do I specify which machine to install the ELK server on versus which to install Filebeat on?_  you can create and edit collections of hosts within the hosts file. Then add the name of your collection of hosts to your playbook file. /etc/ansible/hosts
- _Which URL do you navigate to in order to check that the ELK server is running?
HTTP://http://52.183.69.1:5601/app/kibana
_As a **Bonus**, provide the specific commands the user will need to run to download the playbook, update the files, etc._
