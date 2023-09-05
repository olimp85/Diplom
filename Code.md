# terraform

Я не разделял конфиг на разные задачи и сделал все одним файлом который находится по пути /home/terraform/config.tf

```python
terraform {
  required_providers {
    yandex = {
      source = "yandex-cloud/yandex"
    }
  }
  required_version = ">= 0.13"
}

provider "yandex" {
  cloud_id  = "b1g30bbjrb64gs4sf3l3"
  folder_id = "b1g8815ms561vhvbjsv8"
  zone      = "ru-central1-a"
}

resource "yandex_vpc_network" "olimp85" {
  name = "olimp85"
}

resource "yandex_compute_instance" "vm2" {
  name = "web1"

  resources {
    cores  = 2
    memory = 8
  }

  boot_disk {
    initialize_params {
      image_id = "fd843htdp8usqsiji0bb"
    }
  }

  network_interface {
    subnet_id = yandex_vpc_subnet.net1.id
    nat       = true
  }

  metadata = {
    user-data = "${file("/home/teraform/metadata.txt")}"
  }
}


resource "yandex_vpc_subnet" "net1" {
  name           = "net1"
  zone           = "ru-central1-a"
  network_id     = yandex_vpc_network.olimp85.id
  v4_cidr_blocks = ["192.168.2.0/24"]
}

output "internal_ip_address_vm_2" {
  value = yandex_compute_instance.vm2.network_interface.0.ip_address
}

resource "yandex_compute_instance" "vm3" {
  name = "web2"
  zone = "ru-central1-b"
  resources {
    cores  = 2
    memory = 8
   }

  boot_disk {
    initialize_params {
      image_id = "fd843htdp8usqsiji0bb"
    }
  }

  network_interface {
    subnet_id = yandex_vpc_subnet.net2.id
    nat       = true
  }

  metadata = {
    user-data = "${file("/home/teraform/metadata.txt")}"
  }
}


resource "yandex_vpc_subnet" "net2" {
  name           = "net2"
  zone           = "ru-central1-b"
  network_id     = yandex_vpc_network.olimp85.id
  v4_cidr_blocks = ["192.168.3.0/24"]
}

output "internal_ip_address_vm_3" {
  value = yandex_compute_instance.vm3.network_interface.0.ip_address
}



resource "yandex_compute_instance" "vm6" {
  name = "elasticsearch"
    resources {
    cores  = 2
    memory = 8
   }

  boot_disk {
    initialize_params {
      image_id = "fd843htdp8usqsiji0bb"
    }
  }

  network_interface {
    subnet_id = yandex_vpc_subnet.net1.id
    nat       = true
  }

  metadata = {
    user-data = "${file("/home/teraform/metadata.txt")}"
  }
}
output "internal_ip_address_vm_6" {
  value = yandex_compute_instance.vm6.network_interface.0.ip_address
}

resource "yandex_compute_instance" "vm7" {
  name = "kibana"
  allow_stopping_for_update = true

  resources {
    cores  = 2
    memory = 8
  }

  boot_disk {
    initialize_params {
      image_id = "fd843htdp8usqsiji0bb"
    }
  }

  network_interface {
    subnet_id = yandex_vpc_subnet.net1.id
    nat       = true
    security_group_ids = [yandex_vpc_security_group.olimpsec.id]
}
  metadata = {
    user-data = "${file("/home/teraform/metadata.txt")}"
  }
}


output "internal_ip_address_vm_7" {
  value = yandex_compute_instance.vm7.network_interface.0.ip_address
}


output "external_ip_address_vm_7" {
  value = yandex_compute_instance.vm7.network_interface.0.nat_ip_address
}


resource "yandex_compute_instance" "vm8" {
  name = "bastion"
  allow_stopping_for_update = true

  resources {
    cores  = 2
    memory = 8
  }

  boot_disk {
    initialize_params {
      image_id = "fd843htdp8usqsiji0bb"
    }
  }

  network_interface {
    subnet_id = yandex_vpc_subnet.net1.id
    nat       = true
    security_group_ids =  [yandex_vpc_security_group.olimpbas.id]
  }
  metadata = {
    user-data = "${file("/home/teraform/metadata.txt")}"
  }
}


output "internal_ip_address_vm_8" {
  value = yandex_compute_instance.vm8.network_interface.0.ip_address
}


output "external_ip_address_vm_8" {
  value = yandex_compute_instance.vm8.network_interface.0.nat_ip_address
}


resource "yandex_compute_instance" "vm4" {
  name = "zabbix"

  resources {
    cores  = 2
    memory = 8
  }

  boot_disk {
    initialize_params {
      image_id = "fd843htdp8usqsiji0bb"
    }
  }

  network_interface {
    subnet_id = yandex_vpc_subnet.net1.id
    nat       = true
    security_group_ids = [yandex_vpc_security_group.olimpsec.id]

  }
  metadata = {
    user-data = "${file("/home/teraform/metadata.txt")}"
  }
}
output "internal_ip_address_vm_4" {
  value = yandex_compute_instance.vm4.network_interface.0.ip_address
}


resource "yandex_alb_target_group" "target" {
  name           = "web"

  target {
    subnet_id    = yandex_vpc_subnet.net1.id
    ip_address   = yandex_compute_instance.vm2.network_interface.0.ip_address
  }

  target {
    subnet_id    = yandex_vpc_subnet.net2.id
    ip_address   = yandex_compute_instance.vm3.network_interface.0.ip_address
  }
}


resource "yandex_alb_backend_group" "backend-group" {
  name                     = "backend-group"
  session_affinity {
    connection {
      source_ip = false
    }
  }

  http_backend {
    name                   = "backend-group"
    weight                 = 1
    port                   = 80
    target_group_ids       = [yandex_alb_target_group.target.id]
    load_balancing_config {
      panic_threshold      = 90
    }
    healthcheck {
      timeout              = "10s"
      interval             = "2s"
      healthy_threshold    = 10
      unhealthy_threshold  = 15
      http_healthcheck {
        path               = "/"
      }
    }
  }
}


resource "yandex_alb_http_router" "tf-router" {
  name          = "router"
  labels        = {
    tf-label    = "tf-label-value"
    empty-label = ""
  }
}

resource "yandex_alb_virtual_host" "my-virtual-host" {
  name                    = "vm-main"
  http_router_id          = yandex_alb_http_router.tf-router.id
  route {
    name                  = "main"
    http_route {
      http_route_action {
        backend_group_id  = yandex_alb_backend_group.backend-group.id
        timeout           = "60s"
      }
    }
  }
}

resource "yandex_alb_load_balancer" "test-balancer" {
  name        = "balancer"
  network_id  = yandex_vpc_network.olimp85.id

  allocation_policy {
    location {
      zone_id   = "ru-central1-a"
      subnet_id = yandex_vpc_subnet.net1.id
    }
  }

  listener {
    name = "listener"
    endpoint {
      address {
        external_ipv4_address {
        }
      }
      ports = [ 80 ]
    }
    http {
      handler {
        http_router_id = yandex_alb_http_router.tf-router.id
      }
    }
  }
}


resource "yandex_vpc_security_group" "olimpsec" {
  name        = "olimpsec"
  description = "Description for security group"
  network_id  = yandex_vpc_network.olimp85.id


ingress {
    protocol       = "TCP"
    description    = "kibana"
    v4_cidr_blocks = ["0.0.0.0/0"]
    port           = 5601
  }

ingress {
    protocol       = "TCP"
    description    = "application load balancer"
    v4_cidr_blocks = ["0.0.0.0/0"]
    port           = 80
}

ingress {
    protocol       = "TCP"
    description    = "SSH-permission"
    v4_cidr_blocks = ["192.168.2.0/24"]
    port           = 22
  }

  egress {
    protocol       = "ANY"
    description    = "Rule description 2"
    v4_cidr_blocks = ["0.0.0.0/0"]
    from_port      = 0
    to_port        = 65535
  }
}

resource "yandex_vpc_security_group" "olimpbas" {
  name        = "olimpbas"
  description = "Description for security group"
  network_id  = yandex_vpc_network.olimp85.id

  ingress {
    protocol       = "TCP"
    description    = "bastion"
    v4_cidr_blocks = ["0.0.0.0/0"]
    port           = 22
  }

  egress {
    protocol       = "ANY"
    description    = "Rule description 2"
    v4_cidr_blocks = ["0.0.0.0/0"]
    from_port      = 0
    to_port        = 65535
  }
}


resource "yandex_compute_snapshot_schedule" "default" {
  name = "snapshot"

  schedule_policy {
    expression = "0 9 * * *"
  }

  snapshot_count = 7

  disk_ids = [yandex_compute_instance.vm2.boot_disk[0].disk_id,
              yandex_compute_instance.vm3.boot_disk[0].disk_id,
              yandex_compute_instance.vm4.boot_disk[0].disk_id,
              yandex_compute_instance.vm8.boot_disk[0].disk_id,
              yandex_compute_instance.vm6.boot_disk[0].disk_id,
              yandex_compute_instance.vm7.boot_disk[0].disk_id,
             ]

}
```

# ansible

***bastion***                  

nano /etc/ansible/roles/bastion/tasks/main.yml

```python
---

- name: Copy id_rsa
  copy:
    src: /home/vm1/.ssh/id_rsa
    dest: /home/user/.ssh
    owner: user
    group: user
    mode: '0600'
```

***Elasticsearch***

nano /etc/ansible/roles/Elasticsearch/tasks/main.yml

```python
---

- name: Create User elasticsearch
  user:
    name: elasticsearch
    create_home: no
    shell: /bin/false

- name: Create directories for elasticsearch
  file:
    path: "/tmp/elasticsearch"
    state: directory

- name: Download elasticsearch
  copy:
    src: "/etc/ansible/roles/Elasticsearch/static/elasticsearch-8.8.2-amd64.deb"
    dest: /tmp/elasticsearch

- name: Install java
  apt:
    name=default-jre
    state=latest

- name: Install elasticsearch
  apt:
    deb: "/tmp/elasticsearch/elasticsearch-8.8.2-amd64.deb"

- name: Copy CA to ansible
  ansible.builtin.fetch:
    src: /etc/elasticsearch/certs/http_ca.crt
    dest: /etc/ansible/roles/Elasticsearch/static/
```

***filebeat***

nano /etc/ansible/roles/filebeat/tasks/main.yml

```python
---
- name: Create directories for filebeat
  file:
    path: "/tmp/filebeat"
    state: directory

- name: Download filebeat
  copy:
    src: "/etc/ansible/roles/filebeat/static/filebeat-8.5.2-amd64.deb"
    dest: /tmp/filebeat


- name: Install filebeat
  apt:
    deb: "/tmp/filebeat/filebeat-8.5.2-amd64.deb"

- name: Copy template
  copy:
    src: "/etc/ansible/roles/filebeat/templates/filebeat.yml"
    dest: /etc/filebeat

- name: Copy module
  copy:
    src: "/etc/ansible/roles/filebeat/templates/nginx.yml"
    dest: /etc/filebeat/modules.d/

- name: Copy ca
  copy:
    src: "/etc/ansible/roles/kibana/static/http_ca.crt"
    dest: /etc/filebeat
```

nano /etc/ansible/roles/filebeat/templates/filebeat.yml

```python

---

filebeat.config.modules:
  enabled: true
  path: /etc/filebeat/modules.d/*.yml

output.elasticsearch:
  hosts: ["https://192.168.2.4:9200"]
  username: "elastic"
  password: "gJ56Irber-Jh8QOio=YS"
  ssl:
    enabled: true
    certificate_authorities: ["/etc/filebeat/http_ca.crt"]
```

nano /etc/ansible/roles/filebeat/templates/nginx.yml

```python
- module: nginx
  access:
    enabled: true
    var.paths: ["/var/log/nginx/access.log*"]
  error:
    enabled: true
    var.paths: ["/var/log/nginx/error.log*"]
```

***kibana***

nano /etc/ansible/roles/kibana/tasks/kibana

```python
---

- name: Create directories for kibana
  file:
    path: "/tmp/kibana"
    state: directory

- name: Download kibana
  copy:
    src: "/etc/ansible/roles/kibana/static/kibana-8.8.2-amd64.deb"
    dest: /tmp/kibana

- name: Install kibana
  apt:
    deb: "/tmp/kibana/kibana-8.8.2-amd64.deb"

- name: Copy template
  copy:
    src: "/etc/ansible/roles/kibana/templates/kibana.yml"
    dest: /etc/kibana

- name: Create directories for ca
  file:
    path: "/etc/kibana/certs/"
    state: directory

- name: Copy ca
  copy:
    src: "/etc/ansible/roles/kibana/static/http_ca.crt"
    dest: /etc/kibana/certs/
```

  ***nginx***

  nano /etc/ansible/roles/nginx/tasks/main.yml

```python
  ---
- name: Install Nginx Web Server on Debian Family
  apt:
    name=nginx
    state=latest
  when:
    ansible_os_family == "Debian"
  notify:
    - nginx systemd

- name: Replace nginx.conf
  template:
    src=templates/nginx.conf
    dest=/etc/nginx/nginx.conf

- name: Create home directory
  file:
    path: /var/lib/www
    state: directory

- name: copy the nginx config file and restart nginx
  copy:
    src: /etc/ansible/roles/nginx/templates/static_site.cfg
    dest: /etc/nginx/sites-available/static_site.cfg

- name: create symlink
  file:
    src: /etc/nginx/sites-available/static_site.cfg
    dest: /etc/nginx/sites-enabled/default
    state: link
#    become: true

- name: copy the content of the web site
  copy:
    src: /etc/ansible/roles/nginx/static/
    dest: /var/lib/www
```

***zabbix***

nano /etc/ansible/roles/zabbix/tasks/main.yml

```python

---
- name: Install a .deb package from the internet
  ansible.builtin.apt:
    deb: https://repo.zabbix.com/zabbix/6.0/debian/pool/main/z/zabbix-release/zabbix-release_6.0-4+debian11_all.deb
    update_cache: yes

- name: Install a list of packages
  ansible.builtin.apt:
    pkg:
    - zabbix-server-pgsql
    - zabbix-frontend-php
    - php7.4-pgsql
    - zabbix-apache-conf
    - zabbix-sql-scripts
    - zabbix-agent
    - python3-psycopg2

- name: Create zabbix user
  become: true
  become_user: postgres
  community.postgresql.postgresql_user:
    name: "zabbix"
    password: "qwer1Qwert"
    role_attr_flags: SUPERUSER

- name: Create a new database with name "zabbix"
  become: true
  become_user: postgres
  community.postgresql.postgresql_db:
    name: zabbix
    owner: zabbix

- name: Load dump in DB "zabbix"
  become: true
  become_user: root
  shell: zcat /usr/share/zabbix-sql-scripts/postgresql/server.sql.gz | sudo -u zabbix psql zabbix

- name: Insert after regex, backup, and validate
  blockinfile:
    path: /etc/zabbix/zabbix_server.conf
    backup: yes
    marker: "# {mark} ANSIBLE MANAGED BLOCK "
    insertbefore: '# DBPassword='
    block: |
      DBPassword=qwer1Qwert

- name: zabbix-server
  systemd:
    name: zabbix-server
    state: restarted
```

***zabbix-agent***

nano /etc/ansible/roles/zabbix_agent/tasks/main.yml

```python

---

- name: Install a .deb package from the internet
  ansible.builtin.apt:
    deb: https://repo.zabbix.com/zabbix/6.0/debian/pool/main/z/zabbix-release/zabbix-release_6.0-4+debian11_all.deb
    update_cache: yes

- name: Install a list of packages
  ansible.builtin.apt:
    pkg:
    - zabbix-agent

- name: Insert after regex, backup, and validate
  blockinfile:
    path: /etc/zabbix/zabbix_agentd.conf
    backup: yes
    marker: "# {mark} ANSIBLE MANAGED BLOCK "
    insertbefore: '# Server='
    block: |
      Server=192.168.2.37

- name: zabbix-agent reload
  systemd:
    name: zabbix-agent
    state: restarted
```

***ansible/play.yml***


nano /etc/ansible/play.yml

```python
---
- hosts: all
  become: yes
  become_method:
    sudo
  tasks:
    - name: "Update cache & Full system update"
      apt:
        update_cache: true
        upgrade: dist
        cache_valid_time: 3600
        force_apt_get: true

- hosts: web_servers
  become:
    true
  become_method:
    sudo
  become_user:
    root
  remote_user:
    user
  roles:
   - role: nginx
   - role: filebeat
     tags: filebeat

  vars:
    nginx_user: www-data

- hosts: Elasticsearch
  user: user
  become: true
  become_method: sudo
  become_user: root
  roles:
    - role: Elasticsearch
      tags: elastic

- hosts: kibana
  user: user
  become: true
  become_method: sudo
  become_user: root
  roles:
    - role: kibana
      tags: kibana

- hosts: bastion
  user: user
  become: true
  become_method: sudo
  become_user: root
  roles:
    - role: bastion
      tags: bastion

- hosts: zabbix
  user: user
  become: true
  become_method: sudo
  become_user: root
  roles:
    - role: zabbix
      tags: zabbix

- hosts: web_servers
  user: user
  become: true
  become_method: sudo
  become_user: root
  roles:
    - role: zabbix_agent
      tags: zabbix_agent
