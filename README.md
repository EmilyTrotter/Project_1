# Project_1
Project 1 ELK Stack for Cybersecurity Bootcamp
## Automated ELK Stack Deployment

The files in this repository were used to configure the network depicted below.

! https://github.com/EmilyTrotter/Project_1/tree/main/Diagrams/Emily's Azure Network Architecture 2.0.draw.io.png

These files have been tested and used to generate a live ELK deployment on Azure. They can be used to either recreate the entire deployment pictured above. Alternatively, select portions of the playbook file may be used to install only certain pieces of it, such as Filebeat.

---
- name: Configure ELK-VM with Docker
  hosts: elk
  remote user: azadmin
  become: true
  tasks:
    - name: Install docker.io
      apt:
        update_cache: yes
        force_apt_get: yes
        name: docker.io
        state: present

    - name: Install python3-pip3
      apt:
        force_apt_get: yes
        name: python3-pip
        state: present

    - name: Install Docker module
      pip:
        name: docker
        state: present

    - name: use more memory
      sysctl:
        name: vm.max_map_count
        value: "262144"
        state: present
        reload: yes

    - name: download and launch a docker elk container
      docker_container:
        name: elk
        image: sebp/elk:761
        state: started
        restart policy: always
        published_ports:
          - 5601:5601
          - 9200:9200
          - 5044:5044

    - name: Enable docker service on boot
      systemd:
        name: docker.service
        enabled: yes


---
- name: Installing and Launch Filebeat
  hosts: webservers
  become: yes
  tasks:
 
  - name: Download filebeat .deb file
    command: curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.4.0-amd64.deb

  - name: Install filebeat .deb
    command: dpkg -i filebeat-7.4.0-amd64.deb

  - name: Drop in filebeat.yml
    copy:
      src: /etc/ansible/filebeat-config.yml
      dest: /etc/filebeat/filebeat.yml

  - name: Enable and Configure System Module
    command: filebeat modules enable system

  - name: Setup filebeat
    command: filebeat setup

  - name: Start filebeat service
    command: service filebeat start

  - name: Enable service filebeat on boot
    systemd:
      name: filebeat
      enabled: yes

---
- name: Install metric beat
  hosts: webservers
  become: true
  tasks:

  - name: Download metricbeat
    command: curl -L -O https://artifacts.elastic.co/downloads/beats/metricbeat/metricbeat-7.4.0-amd64.deb

  - name: install metricbeat
    command: dpkg -i metricbeat-7.4.0-amd64.deb

  - name: drop in metricbeat config
    copy:
      src: /etc/ansible/metricbeat-config.yml
      dest: /etc/metricbeat/metricbeat.yml

  - name: enable and configure docker module for metric beat
    command: metricbeat modules enable docker

  - name: setup metric beat
    command: metricbeat setup

  - name: start metric beat
    command: service metricbeat start

  - name: Enable service metricbeat on boot
    systemd:
      name: metricbeat
      enabled: yes



This document contains the following details:
- Description of the Topology
- Access Policies
- ELK Configuration
  - Beats in Use
  - Machines Being Monitored
- How to Use the Ansible Build


### Description of the Topology

The main purpose of this network is to expose a load-balanced and monitored instance of DVWA, the D*mn Vulnerable Web Application.

Load balancing ensures that the application will be highly available, in addition to restricting traffic to the network.
- Load balancers protect the availability of servers. The advantage to a jump box is that it is exposed to the internet while the other machines are hidden behind it, so the jump box just forwards traffic to them after checking security requirements.

Integrating an ELK server allows users to easily monitor the vulnerable VMs for changes to the logs and system resources.
- Filebeat watches for log events/log files and sends them to Kibana.
- Metricbeat gathers information about the processes taking place on the system, how it affects the resources, and sends them to Kibana to be visualized in graphs,tables, etc.

The configuration details of each machine may be found below.

| Name     	| Function 	| IP Address 	| OS    	|
|----------	|----------	|------------	|-------	|
| Jump Box 	| Gateway  	| 10.0.0.4   	| Linux 	|
| Web-1    	| VM       	| 10.0.0.5   	| Linux 	|
| Web-2    	| VM       	| 10.0.0.6   	| Linux 	|
| ELK-VM   	| VM       	| 10.1.0.4   	| Linux 	|

### Access Policies

The machines on the internal network are not exposed to the public Internet. 

Only the jump box machine can accept connections from the Internet. Access to this machine is only allowed from the following IP addresses:
73.11.4.198

Machines within the network can only be accessed by the Ansible container within the jump box.
- The jump box was able to access the ELK-VM. The address for the jump box provisioner is 10.0.0.4 (private) and 104.45.135.73 (public).

A summary of the access policies in place can be found in the table below.

| NSG        	| Priority 	| Name             	| Port 	| Protocol 	| Source      	| Destination    	|
|------------	|----------	|------------------	|------	|----------	|-------------	|----------------	|
| RedTeamSec 	| 500      	| Jump-BoxSSH      	| 22   	| TCP      	| 10.0.0.4    	| VirtualNetwork 	|
| RedTeamSec 	| 520      	| AllowMyIPSSH     	| 22   	| TCP      	| 73.11.4.198 	| VirtualNetwork 	|
| RedTeamSec 	| 530      	| Port_80          	| 80   	| Any      	| 73.11.4.198 	| VirtualNetwork 	|
| ELK-VM-nsg 	| 500      	| SSHFromJumpbox   	| 22   	| TCP      	| 10.0.0.4    	| VirtualNetwork 	|
| ELK-VM-nsg 	| 510      	| AllowMyIPTraffic 	| 5601 	| TCP      	| 73.11.4.198 	| VirtualNetwork 	|

### Elk Configuration

Ansible was used to automate configuration of the ELK machine. No configuration was performed manually, which is advantageous because it speeds up the process and eliminates more risk of human error.

The playbook implements the following tasks:
- The first three tasks are to install docker.io in order to download the container, install pip3 (the package manager), and finally to install the docker python module
- The next step was to increase the memory of the elk container
- After that came the downloading and launching of the elk container with its specified ports
- Lastly, the systemd module was used to ensure the service always starts upon booting

The following screenshot displays the result of running `docker ps` after successfully configuring the ELK instance.

! https://github.com/EmilyTrotter/Project_1/tree/main/IMAGES/Elk_container_ps.png

### Target Machines & Beats
This ELK server is configured to monitor the following machines:
- 10.0.0.5 Web-1
- 10.0.0.6 Web-2
- 10.1.0.4 ELK-VM

We have installed the following Beats on these machines:
- Filebeat
- Metricbeat

These Beats allow us to collect the following information from each machine:
- In this project, Filebeat and Metricbeat were downloaded to the VMs to collect data. Filebeat collects log data, such as login attempts. Metricbeat collects measurements of resources being used on a machine; an example of this would be measuring CPU usage.

### Using the Playbook
In order to use the playbook, you will need to have an Ansible control node already configured. Assuming you have such a control node provisioned: 

SSH into the control node and follow the steps below:
- Copy the configuration file to /etc/filebeat/filebeat.yml or /etc/metricbeat/metricbeat.yml.
- Update the configuration file to include the desired host IP address, which is 10.1.0.4 for the ELK-VM.
- Run the playbook, and navigate to Kibana to check that the installation worked as expected.

Answer the following questions to fill in the blanks:
Which file is the playbook? Where do you copy it?
- filebeat-playbook.yml and metricbeat-playbook.yml. They are copied to /etc/filebeat/filebeat.yml and /etc/metricbeat/metricbeat.yml, this information is found under the "drop in" task headers.
Which file do you update to make Ansible run the playbook on a specific machine? How do I specify which machine to install the ELK server on versus which to install Filebeat on?
-The playbook.yml file, specifically the host line, needs to be updated in order to run the playbook on a specific machine. For example, in the filebeat-playbook.yml, the hosts line specifies which machines you want the playbook to run on--which is "webservers" in this case. The ELK server was downloaded onto the ELK-VM by having its own playbook created, in which the host specified was "elk"; the same is true with filebeat with the hosts being "webservers".
Which URL do you navigate to in order to check that the ELK server is running? 
- http://13.78.181.142/app/kibana
