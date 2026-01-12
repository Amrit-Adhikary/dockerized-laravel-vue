# CloudTech DevOps Pre-Screening Task — Setup Guide

This guide explains how to set up the **Laravel backend** and **Vue.js frontend** using Docker.  
It is beginner-friendly and designed for **one-copy-paste execution**.

---

## 🚀 Prerequisites

Make sure your system has:

- Docker & Docker Compose  
- Node.js & npm  
- Composer  
- Git  

Install on Ubuntu/Debian:

```bash
sudo apt update
sudo apt install docker.io docker-compose git npm curl unzip composer -y
sudo systemctl enable docker
sudo systemctl start docker

Verify:

docker --version
docker compose version
npm --version
composer --version

📂 Project Structure

cloudtech-devops-task/
│
├── backend-laravel/
│   ├── Dockerfile
│   ├── docker-compose.yml
│   ├── docker/nginx/default.conf
│   └── src/           # Laravel project
│
├── frontend-vue/
│   └── vue-app/
│       ├── Dockerfile
│       ├── package.json
│       └── src/       # Vue project
│
├── run_all.sh         # One-command run script
└── Setup_Guide.md     # This file

🏗 Part 1 — Backend (Laravel + Nginx + MySQL)
1. Go to backend folder

cd ~/cloudtech-devops-task/backend-laravel

2. Create Laravel project

composer create-project laravel/laravel src

3. Create Docker & Nginx folders

mkdir -p docker/nginx
touch Dockerfile docker-compose.yml docker/nginx/default.conf

4. Nginx Config (docker/nginx/default.conf)

server {
    listen 80;
    root /var/www/public;
    index index.php index.html;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        fastcgi_pass app:9000;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    }
}

5. Dockerfile (Dockerfile)

FROM php:8.2-fpm

RUN apt-get update && apt-get install -y \
    libpng-dev zip unzip git curl \
    && docker-php-ext-install pdo pdo_mysql

WORKDIR /var/www
COPY src/ .

RUN chown -R www-data:www-data /var/www

6. Docker Compose (docker-compose.yml)

services:
  app:
    build: .
    container_name: laravel_app
    volumes:
      - ./src:/var/www

  nginx:
    image: nginx:latest
    container_name: laravel_nginx
    ports:
      - "8080:80"
    volumes:
      - ./src:/var/www
      - ./docker/nginx/default.conf:/etc/nginx/conf.d/default.conf
    depends_on:
      - app

  db:
    image: mysql:8.0
    container_name: laravel_db
    environment:
      MYSQL_DATABASE: laravel
      MYSQL_ROOT_PASSWORD: root
    ports:
      - "3306:3306"

7. Run backend containers

docker compose up -d --build

Access in browser: http://localhost:8080
🎨 Part 2 — Frontend (Vue.js)
1. Navigate to frontend folder

cd ~/cloudtech-devops-task/frontend-vue

2. Create Vue project

npm create vue@latest

Prompts:

    Project name: vue-app

    Add TypeScript? ❌ No

    Add Router? ✅ Yes

    Add Pinia? ❌ No

    Add Vitest? ❌ No

    Add ESLint? ❌ No

cd vue-app
npm install

3. Dockerfile for Vue (Dockerfile)

# Stage 1: Build Vue app
FROM node:20-alpine as build
WORKDIR /app
COPY . .
RUN npm install
RUN npm run build

# Stage 2: Serve with Nginx
FROM nginx:alpine
COPY --from=build /app/dist /usr/share/nginx/html
EXPOSE 80

4. Build and run Vue container

docker build -t vue-app .
docker run -d -p 3000:80 vue-app

Access in browser: http://localhost:3000
⚡ Part 3 — One-command Run Script

Create run_all.sh in project root:

cd ~/cloudtech-devops-task
nano run_all.sh

Paste:

#!/bin/bash
# Run both backend and frontend

echo "✅ Starting Laravel Backend..."
cd backend-laravel || exit
docker compose up -d --build
echo "✅ Backend: http://localhost:8080"

cd ../frontend-vue/vue-app || exit
echo "✅ Starting Vue Frontend..."
docker build -t vue-app .
docker run -d -p 3000:80 vue-app
echo "✅ Frontend: http://localhost:3000"
echo "🚀 All services are up!"

Make executable:

chmod +x run_all.sh

Run anytime with:

./run_all.sh

