main.tf
terraform {
  required_providers {
    docker = {
      source  = "kreuzwerker/docker"
      version = "~> 3.0.1"
    }
  }
}

provider "docker" {}

resource "docker_image" "nginx" {
  name         = "nginx"
  keep_locally = false
}

resource "docker_container" "nginx" {
  image = docker_image.nginx.image_id
  name  = "tutorial"

  ports {
    internal = 80
    external = 8080
  }
}

# New resource with dependencies
resource "null_resource" "example" {
  # Explicit dependency on the docker_container.nginx resource
  depends_on = [docker_container.nginx]

  # Implicit dependency on the docker_image.nginx resource
  triggers = {
    container_id = docker_container.nginx.id
  }

  # Dummy provisioner for demonstration purposes
  provisioner "local-exec" {
    command = "echo This resource depends on the nginx Docker container and image"
  }
}
