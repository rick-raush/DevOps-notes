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

# Buildx

Build & push in **one step** for multiple platforms:

```bash
docker buildx build \
  -f Dockerfile \
  -t 337891198417.dkr.ecr.ap-south-1.amazonaws.com/digitaldevops:cortex-agent-8.3-updated-manifest \
  --platform linux/amd64,linux/arm64 \
  --push .
```

✅ Explanation:

- `--platform linux/amd64,linux/arm64` → Builds for both x86_64 and ARM64.
    
- `--push` → Pushes directly to ECR after building; no local image stored.
    
- `-f Dockerfile` → Path to your Dockerfile.
    

`t ...` → Image name with tag.

Absolutely! Let’s break down this **Docker Buildx command** step by step and compare it to the traditional Docker build/push workflow.

* * *

## **1️⃣ The command explained**

```bash
docker buildx build \
  -f Dockerfile \
  -t 337891198417.dkr.ecr.ap-south-1.amazonaws.com/digitaldevops:cortex-agent-8.3-updated-manifest \
  --platform linux/amd64,linux/arm64 \
  --push .
```

**Breakdown:**

| Part | Meaning |
| --- | --- |
| `docker buildx build` | Uses **Docker Buildx**, which is an extended builder for Docker that supports **multi-platform builds**, **buildkit**, and advanced features. |
| `-f Dockerfile` | Specifies the Dockerfile to use. If omitted, it defaults to `./Dockerfile`. |
| `-t 337891198417.dkr.ecr.ap-south-1.amazonaws.com/digitaldevops:cortex-agent-8.3-updated-manifest` | Tags the image with your **ECR repository name** and tag. |
| `--platform linux/amd64,linux/arm64` | Builds the image for **multiple CPU architectures**: x86_64 (amd64) and ARM64. |
| `--push` | Pushes the resulting image directly to **ECR**, skipping local storage. |
| `.` | Context for the build, i.e., current directory with Dockerfile and files. |

* * *

## **2️⃣ How it differs from traditional Docker build**

### **Traditional Docker build + push**

```bash
docker build -f Dockerfile -t my-image:tag .
docker tag my-image:tag <ecr-repo-url>:tag
docker push <ecr-repo-url>:tag
```

- Builds for **a single platform** (usually the one your Docker daemon runs on).
    
- Image is first stored locally and then pushed.
    
- Multi-arch images are not supported without extra steps.
    

* * *

### **Docker Buildx advantages**

1.  **Multi-platform builds**
    
    - Build one image for **x86_64, ARM64**, etc., in a single command.
        
    - Useful for CI/CD pipelines targeting different architectures (AWS Graviton, Mac M1/M2, etc.).
        
2.  **Direct push**
    
    - `--push` uploads the image immediately to the registry without storing it locally.
3.  **BuildKit support**
    
    - Faster builds with caching, parallel steps, and smaller final images.
4.  **Manifest creation**
    
    - Automatically creates a **multi-arch manifest** in the registry, so pulling the image automatically chooses the correct architecture for the host machine.

* * *

### **3️⃣ Key takeaway**

- **Traditional Docker build** → Single-platform, stored locally first, then pushed.
    
- **Buildx** → Multi-platform, uses BuildKit, can push directly, and automatically handles manifests for ARM/x86.
    

* * *

&nbsp;