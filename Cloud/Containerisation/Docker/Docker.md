---
tags: [cloud, containerisation, docker]
type: cheatsheet
source: personal Notion — "Commands Master Cheatsheet" (Docker & Containerization section)
last-verified: 2026-08-25
---

# Docker

## Up
- [[Containerisation]]

Command cheatsheet for Docker and Docker Compose, imported from my Notion master cheatsheet.

## Subtopics
- [[Lazydocker]] — terminal UI for Docker
- [[Watchtower]] — automatic container image updates
- [[Podman]] — daemonless, rootless container engine
- [[BuildKit & buildx]] — modern build engine (cache, secrets, multi-arch)
- [[Dockerfile Best Practices]] — fast, small, secure images
- [[Harbor]] — private registry (RBAC, scanning, signing)

## Docker Basics
### Images
| Command | Description |
|---|---|
| `docker images` | List images |
| `docker pull image:tag` | Pull image |
| `docker rmi image` | Remove image |
| `docker build -t name .` | Build from Dockerfile |
### Containers
| Command                                  | Description                  |
| ---------------------------------------- | ---------------------------- |
| `docker ps`                              | List running containers      |
| `docker ps -a`                           | List all containers          |
| `docker run image`                       | Run container                |
| `docker run -it image /bin/bash`         | Interactive shell            |
| `docker run -d image`                    | Detached mode                |
| `docker run -p 8080:80 image`            | Port mapping                 |
| `docker run -v /host:/container image`   | Volume mount                 |
| `docker stop container_id`               | Stop container               |
| `docker start container_id`              | Start container              |
| `docker restart container_id`            | Restart container            |
| `docker rm container_id`                 | Remove container             |
| `docker exec -it container_id /bin/bash` | Execute in running container |
### Logs and Info
| Command | Description |
|---|---|
| `docker logs container_id` | View logs |
| `docker logs -f container_id` | Follow logs |
| `docker inspect container_id` | Detailed info |
| `docker stats` | Resource usage |
### Networks
| Command | Description |
|---|---|
| `docker network ls` | List networks |
| `docker network create name` | Create network |
| `docker network connect network container` | Connect container to network |
### Volumes
| Command | Description |
|---|---|
| `docker volume ls` | List volumes |
| `docker volume create name` | Create volume |
| `docker volume rm name` | Remove volume |
### Clean Up
| Command | Description |
|---|---|
| `docker system prune` | Remove unused data |
| `docker container prune` | Remove stopped containers |
| `docker image prune` | Remove unused images |
## Docker Compose
| Command | Description |
|---|---|
| `docker-compose up` | Start services |
| `docker-compose up -d` | Detached mode |
| `docker-compose down` | Stop services |
| `docker-compose down -v` | Remove volumes too |
| `docker-compose logs` | View logs |
| `docker-compose logs -f` | Follow logs |
| `docker-compose logs service` | Logs for specific service |
| `docker-compose exec service command` | Execute command in service |
| `docker-compose build` | Build images |
| `docker-compose up -d --scale service=3` | Scale services |
