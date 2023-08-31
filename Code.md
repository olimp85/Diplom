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
