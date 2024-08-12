--- 
title: Configuring a homelab using Terraform and Ansible on Proxmox and Kubernetes - Part 1
date: 2024-08-06
categories: [Guide, Tutorial]
tags: [ansible, homelab, proxy, dns]
author: jvekeman
pin: false
---

In this series of guides, we will set up a homelab using Proxmox as our hypervisor, Kubernetes as our container orchestration platform, and a Raspberry Pi as DNS and proxy server. To automate the setup Terraform and Ansible will be used. We will start by setting up a DNS and a Proxy on a Raspberry Pi.


# Part 1: Setting up a DNS and Proxy server on a Raspberry Pi

## Prerequisites

Before we start, we need to make sure that we have a Raspberry Pi available. We will be using a Raspberry Pi 4 with 2GB of RAM, but any Raspberry Pi should do. To connect to the Raspberry Pi, we will use SSH.

In addition, we need Ansible installed on a machine. Ansible is used to automate the setup of the Raspberry Pi. For documentation on installing Ansible, please refer to the official [Ansible documentation](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html). We will be using Ansible on a Windows machine (WSL).

> Knowing how to use Ansible is recommended, but not required. The necessary Ansible playbooks and roles to automate the setup of the Raspberry Pi are provided in this guide.
{: .prompt-tip }

## The Ansible environment

### Inventory file

To start, create a file called `inventory.yml` in the root directory of your Ansible project. This file contains the IP address of the Raspberry Pi and the SSH variables.

```yaml
servers:
  vars:
    ansible_user: joren
    # ansible_ssh_pass: password # Uncomment this line if you want to use a password
    ansible_ssh_common_args: '-o StrictHostKeyChecking=no'
    ansible_ssh_private_key_file: '~/.ssh/ansible-key'
    ansible_python_interpreter: /usr/bin/python3
  hosts:
    rpi:
      ansible_host: 192.168.1.31
```

This inventory file contains the IP address of the Raspberry Pi, the SSH user, the SSH private key file and the Python interpreter. The `rpi` host is used in the Ansible playbooks. The `ansible_ssh_common_args: '-o StrictHostKeyChecking=no'` is used to disable the SSH host key checking. This is useful when connecting to a new host for the first time.

> This example uses user `joren` but if wanted you can create a dedicated user for Ansible.
{: .prompt-tip }

### The ansible playbook

Next, an Ansible playbook is needed to automate the setup of the Raspberry Pi. Create a file called `rpi.yml` in the root directory of your Ansible project.

```yaml
# ----------------------------------------------------------
# RPI configuration
# ----------------------------------------------------------
- name: Installing Components
  hosts: rpi
  roles: 
    - basic
    - docker
    - dns
    - proxy
```

This playbook contains the roles that are used to automate the setup of the Raspberry Pi. We will create a role for the basic setup, Docker, DNS and Proxy. Using roles makes our playbook more readable and maintainable. It also allows the reuse in other playbooks/hosts.

## The Ansible roles

Before creating the roles we need to create the directory structure. Start by creating a directory called `roles` in the root directory of our Ansible project. In this directory, create a directory for each role.

After this section the directory structure should look like this:

```txt
roles/
├── basic
│   ├── tasks
│   │   └── main.yml
├── docker
│   ├── tasks
│   │   └── main.yml
├── dns
│   ├── tasks
│   │   └── main.yml
└── proxy
    ├── tasks
    │   └── main.yml
    └── files
        └── nginx.conf
inventory.yml
rpi.yml
```

### The basic role

We start by creating the basic role. This role enables the firewall and allows SSH connections.

In the `roles/basic/tasks/main.yml` file, add the following tasks:

```yaml
---
- name: Enable UFW firewall
  community.general.ufw:
    state: enabled
    policy: deny
  become: yes

- name: Allow SSH port
  community.general.ufw:
     rule: allow
     port: "22" 
  become: yes

- name: Install python3-pip package
  apt:
    name: python3-pip
    state: present
  become: yes
  ignore_errors: yes
```

> Feel free to add more tasks to the basic role if needed.
{: .prompt-tip }

### The docker role

Next, we will create the Docker role. This role installs Docker on the Raspberry Pi and is created by [CrimsonCloak](https://github.com/CrimsonCloak)

In the `roles/docker/tasks/main.yml` file, add the following tasks:

```yaml
---
- name: Gather the facts
  ansible.builtin.setup:

- name: Set Docker repository variables (enkel ubuntu)
  set_fact:
    docker_repo: "deb http://download.docker.com/linux/ubuntu {{ ansible_facts['distribution_release'] | lower }} stable"
  when: ansible_facts['distribution'] == "Ubuntu"

- name: Install required packages
  apt:
    name: 
      - ca-certificates
      - curl
    state: present
  become: yes

- name: Add Docker's official GPG key
  apt_key:
    url: https://download.docker.com/linux/ubuntu/gpg
    state: present
  become: yes

- name: Add Docker repository to apt 
  apt_repository:
    repo: "{{ docker_repo }}"
    state: present
    update_cache: yes
  become: yes

- name: Install Docker
  apt:
    name: 
      - docker-ce
      - docker-ce-cli
      - containerd.io
      - docker-buildx-plugin
      - docker-compose-plugin
    state: present
  become: yes

- name: Add user(s), {{ username }} to Docker group
  user:
    name: "{{ username }}"
    groups: docker
    append: yes
  become: yes

- name: Start and enable Docker
  service:
    name: docker
    state: started
    enabled: yes
  become: yes

- name: Install Docker Python package 
  pip: 
    name: docker 
    state: present
  ignore_errors: yes
  become: yes
```

This role not only installs Docker on the Raspberry Pi but also adds the user to the Docker group and installs the Docker Python package.

### The DNS role

The DNS role installs Adguard Home. Adguard Home is a network-wide software for blocking ads and tracking. It acts as a DNS server and can be used to block ads on your network.

In the `roles/dns/tasks/main.yml` file, add:

```yaml
---
- name: Create resolved.conf.d directory
  file:
    path: /etc/systemd/resolved.conf.d
    state: directory
  become: yes

- name: Create adguardhome.conf file
  copy:
    dest: /etc/systemd/resolved.conf.d/adguardhome.conf
    content: |
      [Resolve]
      DNS=127.0.0.1
      DNSStubListener=no
  become: yes
  register: adguardhome_conf

- name: Create symlink between /run/systemd/resolve/resolv.conf and /etc/resolv.conf 
  file: 
    src: /run/systemd/resolve/resolv.conf
    dest: /etc/resolv.conf
    state: link
  become: yes

- name: Restart systemd-resolved
  systemd:
    name: systemd-resolved
    state: restarted
    daemon_reload: yes
  become: yes
  when: adguardhome_conf.changed

- name: Start AdGuard Home container
  community.docker.docker_container:
    name: adguardhome
    image: adguard/adguardhome
    restart_policy: always
    ports:
      - "53:53/tcp"
      - "53:53/udp"
      - "3000:3000/tcp"
    volumes:
      - "./data/adguard/work:/opt/adguardhome/work"
      - "./data/adguard/conf:/opt/adguardhome/conf"
    networks:
      - name: dns_proxy_network
  register: adguardhome_container
```

First, this role creates a file called `adguardhome.conf` in the `/etc/systemd/resolved.conf.d` directory. Next, it creates a symlink between `/run/systemd/resolve/resolv.conf` and `/etc/resolv.conf`. This is needed because by default systemd-resolved uses port 53. Finally, it creates and starts the Adguard Home container. 

### The Proxy role

The Proxy role installs Nginx and configures it as a reverse proxy. It also installs Certbot to obtain SSL certificates using a DNS challenge (Cloudflare).
Creating a token can be done using this [guide](https://developers.cloudflare.com/fundamentals/api/get-started/create-token/) (Zone:DNS:Edit).

In the `roles/proxy/tasks/main.yml` file, add:

```yaml
- name: install snapd
  apt:
    name: snapd
    state: present
  become: yes
  register: snapd_install

- name: remove certbot-auto and any Certbot OS packages
  apt:
    name: certbot
    state: absent
  become: yes

- name: prepare the Certbot command
  file:
    src: /snap/bin/certbot
    dest: /usr/bin/certbot
    state: link
  become: yes

- name: confirm plugin containment level 
  command: snap set certbot trust-plugin-with-root=ok
  when: snapd_install.changed
  become: yes

- name: Install correct DNS plugin
  command: snap install certbot-dns-cloudflare
  when: snapd_install.changed
  become: yes

- name: create directory for certbot
  file: 
    path: ./.secrets/certbot/
    state: directory

- name: create cloudflare.ini
  copy:
    dest: ./.secrets/certbot/cloudflare.ini
    content: |
      dns_cloudflare_api_token = {{ cloudflare_token }}

- name: get cert for {{ domain }}
  command: certbot certonly --dns-cloudflare --dns-cloudflare-credentials ./.secrets/certbot/cloudflare.ini -d *.{{ domain }} --non-interactive --agree-tos -m "{{ email }}"
  when: snapd_install.changed

- name: create directory for nginx
  file:
    path: ./data/nginx
    state: directory

- name: move nginx conf to correct location
  copy:
    src: ./nginx.conf
    dest: ./data/nginx/nginx.conf

- name: install nginx using docker container
  docker_container:
    name: nginx
    image: nginx:latest
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./data/nginx/nginx.conf:/etc/nginx/nginx.conf
      - /etc/letsencrypt/live/{{ domain }}/fullchain.pem:/etc/nginx/ssl/fullchain.pem
      - /etc/letsencrypt/live/{{ domain }}/privkey.pem:/etc/nginx/ssl/privkey.pem
    restart_policy: always
    networks:
      - name: dns_proxy_network
```

This role first installs Certbot and the Cloudflare DNS plugin. It will then obtain an SSL certificate for the domain using the DNS challenge. Next, it will create a directory for Nginx and move the Nginx configuration file to the correct location. Finally, it will install Nginx using a Docker container.

#### The Nginx configuration

Create a file called `nginx.conf` in `roles/proxy/files/nginx.conf`:

```conf
user nginx;
worker_processes 1;
worker_rlimit_nofile 8192;
events {
    worker_connections 4096;
}
http {
  index    index.html index.htm index.php;
  ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3; # Dropping SSLv3, ref: POODL>  ssl_prefer_server_ciphers on;
  default_type application/octet-stream;
  log_format   main $remote_addr - $remote_user [$time_local]  $status
    $request $body_bytes_sent $http_referer
    $http_user_agent $http_x_forwarded_for;
  sendfile     on;
  tcp_nopush   on;
  server_names_hash_bucket_size 128; # this seems to be required

  ssl_certificate     /etc/nginx/ssl/fullchain.pem;
  ssl_certificate_key /etc/nginx/ssl/privkey.pem;

  server {
    listen 80;
    server_name *.<yourdomain>;
    return 301 https://$host$request_uri;
  }

  server {
    listen 443 ssl;
    http2 on;
    server_name adguard.<yourdomain>;
    location / {
      proxy_pass http://adguardhome:3000/;
    }
  }

}
```

Replace `<yourdomain>` with your domain name.

This configuration file creates a server block for the Adguard Home container. It will listen on port 443 and proxy requests to the Adguard Home container. Requests to port 80 will be redirected to https.

## Running the playbook

Before running the playbook, we need to pass some variables to the roles.

Change the `rpi.yml` file to:

```yaml
---
# ----------------------------------------------------------
# RPI configuration
# ----------------------------------------------------------
- name: Installing Components
  hosts: rpi
  roles: 
    - basic
    - role: docker
      username: "YOUR_USERNAME"
    - dns
    - role: proxy
      domain: "YOUR_DOMAIN"
      email: "YOUR_EMAIL"
      cloudflare_token: "YOUR_CLOUDFLARE_TOKEN"
```

After this, run the playbook using the following command:

```bash
ansible-playbook -i inventory.yml rpi.yml
```

> Instead of placing your Cloudflare token in the playbook, you can use Ansible Vault to encrypt the token. For more information, refer to the [Ansible documentation](https://docs.ansible.com/ansible/latest/user_guide/vault.html).
{: .prompt-tip }

## Notes

- The roles are created for Ubuntu. If you are using a different distribution, you may need to modify the roles.
- Certbot will obtain a Wildcard SSL certificate for the domain using the Cloudflare DNS plugin.
- Before using Adguard Home some installation steps are needed. These show when you visit the IP address of the Raspberry Pi in a browser. (Remeber to set the dasboard port to 3000)
- The Nginx configuration is a basic configuration. You can modify it to suit your needs.
- Adguard Home needs some redirects to the proxy before it works correctly. (Part 2)

## Conclusion

In this guide, we have set up a DNS and Proxy server on a Raspberry Pi using Ansible. We have created an Ansible inventory file, playbook and roles to automate the setup. Certbot was used to obtain an SSL certificate for the domain using the Cloudflare DNS plugin. In the next part, we will provision Adguard Home using Terraform.