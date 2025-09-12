# 🐳 Docker Commands Cheat Sheet

### 🔹 Rule of Thumb

`docker run` syntax is:

`docker run [OPTIONS] IMAGE [COMMAND] [ARG...]`

- `[OPTIONS]` → all flags (`-it`, `--name`, `-p`, `-v`, etc.)
    
- `IMAGE` → the Docker image (`ubuntu`)
    
- `[COMMAND]` → what to run inside (`/bin/bash`, `sleep`, etc.)
    

* * *

## 🎨 **IMAGES**

| Command | 📝 Description |
| --- | --- |
| `docker images` | 📋 List all local images |
| `docker rmi <image_name>` | 🗑️ Delete an image |
| `docker image prune` | 🧹 Remove unused images |
| `docker build -t <image_name>:<version> .` | 🏗️ Build an image from a Dockerfile (version optional) |
| `docker build -t <image_name>:<version> . --no-cache` | 🔄 Build image without cache |

* * *

## 📦 **CONTAINERS**

| Command | 📝 Description |
| --- | --- |
| `docker ps -a` | 📋 List all containers (running & stopped) |
| `docker ps` | ▶️ List running containers |
| `docker run <image_name>` | 🚀 Create & run new container (pulls if not local) |
| `docker run -d <image_name>` | 🎯 Run in background (detached mode) |
| `docker run --name <container_name> <image_name>` | 🏷️ Run with custom name |
| `docker run -p <host_port>:<container_port> <image_name>` | 🔌 Port Binding |
| `docker run -e <var_name>=<var_value> <container_name>` | 🌍 Set environment variables |
| `docker start <container_name>` | ▶️ Start container |
| `docker stop <container_name>` | ⏹️ Stop container |
| `docker inspect <container_name>` | 🔍 Inspect container details |
| `docker rm <container_name>` | 🗑️ Delete a container |

* * *

## 🛠️ **TROUBLESHOOTING**

| Command | 📝 Description |
| --- | --- |
| `docker logs <container_name>` | 📜 View container logs |
| `docker exec -it <container_name> /bin/bash` | 🐚 Open shell (bash) |
| `docker exec -it <container_name> sh` | 🐚 Open shell (sh) |

* * *

## ☁️ **DOCKER HUB**

| Command | 📝 Description |
| --- | --- |
| `docker pull <image_name>` | ⬇️ Pull image from DockerHub |
| `docker push <username>/<image_name>` | ⬆️ Push image to DockerHub |
| `docker login -u <username>` | 🔑 Login to DockerHub |
| `docker logout` | 🚪 Logout from DockerHub |
| `docker search <image_name>` | 🔍 Search images on DockerHub |

* * *

## 💾 **VOLUMES**

| Command | 📝 Description |
| --- | --- |
| `docker volume ls` | 📋 List volumes |
| `docker volume create <volume_name>` | ➕ Create named volume(docker managed) |
| `docker volume rm <volume_name>` | 🗑️ Remove named volume |
| `1.docker run --volume <volume_name>:<mount_path-inContainer>` | 📦 Mount named volume |
| `docker run --mount type=volume,src=<volume_name>,dest=<mount_path>` | 📦 Mount named volume (alt syntax) |
| `2.docker run --volume <mount_path>` | 🌀 Anonymous volume |
| `3.docker run --volume <host_path>:<container_path>` | 🔗 Bind mount (host → container) |
| `docker run --mount type=bind,src=<host_path>,dest=<container_path>` | 🔗 Bind mount (alt syntax) |
| `docker volume prune` | 🧹 Remove unused volumes |

* * *

## 🌐 **NETWORKS**

| Command | 📝 Description |
| --- | --- |
| `docker network ls` | 📋 List networks |
| `docker network create <network_name>` | 🌐 Create a network |
| `docker network rm <network_name>` | 🗑️ Remove a network |
| `docker network prune` | 🧹 Remove unused networks |
| docker network inspect &lt;netName&gt; | Inspect a network: |
| docker network disconnect &lt;netName&gt; &lt;container_id_or_name&gt; | **Disconnect the containers(stopped and rmed)** from that network: |

* * *

&nbsp;

&nbsp;