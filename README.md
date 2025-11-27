🚀 Discover Dollar – DevOps Internship Assignment

Full-stack MEAN Application Deployment with Docker, Nginx, CI/CD (GitHub Actions), and AWS EC2.

📦 Project Overview

This project deploys a complete MEAN stack (MongoDB, Express, Angular, Node.js) application using:

Docker & Docker Compose

Nginx Reverse Proxy

MongoDB Database

AWS EC2 (Ubuntu 24.04)

GitHub Actions CI/CD Pipeline

Docker Hub for image hosting

The application deployment is fully automated:
Whenever code is pushed → GitHub Actions builds containers → pushes images → deploys to EC2 automatically.

🌐 Live Application

Component

URL

Frontend

http://13.203.213.222/

Backend API

http://13.203.213.222/api/tutorials

🧰 Tech Stack

Docker

Docker Compose

Nginx (Reverse Proxy)

MongoDB (Dockerized)

Node.js + Express

Angular

AWS EC2

GitHub Actions (CI/CD)

Docker Hub

📁 Project Structure

/backend
/frontend
docker-compose.yml
nginx.conf
.github/workflows/build-and-deploy.yml


⚙️ Configuration Files

🐳 Docker Compose (docker-compose.yml)

version: '3.8'

services:
  mongo:
    image: mongo:6
    restart: unless-stopped
    volumes:
      - mongo-data:/data/db

  backend:
    image: ${DOCKER_USERNAME}/crud-backend:latest
    restart: unless-stopped
    depends_on:
      - mongo
    environment:
      MONGO_URL: "mongodb://mongo:27017/mydb"
    expose:
      - "3000"

  frontend:
    image: ${DOCKER_USERNAME}/crud-frontend:latest
    restart: unless-stopped
    depends_on:
      - backend
    expose:
      - "80"

  nginx:
    image: nginx:alpine
    restart: unless-stopped
    ports:
      - "80:80"
    depends_on:
      - frontend
      - backend
    volumes:
      - ./nginx.conf:/etc/nginx/conf.d/default.conf:ro

volumes:
  mongo-data:


🧭 Nginx Configuration (nginx.conf)

server {
    listen 80;

    location /api/ {
        proxy_pass http://backend:3000/;
    }

    location / {
        proxy_pass http://frontend:80/;
    }
}


🤖 GitHub Actions Workflow (.github/workflows/build-and-deploy.yml)

name: build-and-deploy

on:
  push:
    branches: ["main"]

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v3

    - name: Login to Docker Hub
      uses: docker/login-action@v2
      with:
        username: ${{ secrets.DOCKERHUB_USERNAME }}
        password: ${{ secrets.DOCKERHUB_PASSWORD }}

    - name: Build backend image
      run: docker build -t "${{ secrets.DOCKERHUB_USERNAME }}/crud-backend:latest" ./backend

    - name: Push backend image
      run: docker push "${{ secrets.DOCKERHUB_USERNAME }}/crud-backend:latest"

    - name: Build frontend image
      run: docker build -t "${{ secrets.DOCKERHUB_USERNAME }}/crud-frontend:latest" ./frontend

    - name: Push frontend image
      run: docker push "${{ secrets.DOCKERHUB_USERNAME }}/crud-frontend:latest"

    - name: Copy docker-compose + nginx.conf to EC2
      uses: appleboy/scp-action@v0.1.4
      with:
        host: ${{ secrets.EC2_HOST }}
        username: ubuntu
        key: ${{ secrets.EC2_SSH_KEY }}
        source: "docker-compose.yml,nginx.conf"
        target: "/home/ubuntu/app"

    - name: SSH into EC2 & restart services
      uses: appleboy/ssh-action@v1.0.3
      with:
        host: ${{ secrets.EC2_HOST }}
        username: ubuntu
        key: ${{ secrets.EC2_SSH_KEY }}
        script: |
          echo "${{ secrets.DOCKERHUB_PASSWORD }}" | sudo docker login -u "${{ secrets.DOCKERHUB_USERNAME }}" --password-stdin
          cd /home/ubuntu/app
          echo "DOCKER_USERNAME=${{ secrets.DOCKERHUB_USERNAME }}" > .env
          sudo docker-compose pull
          sudo docker-compose down
          sudo docker-compose up -d


📸 Screenshots

1. GitHub Repository

All files: backend, frontend, docker-compose.yml, nginx.conf, workflows

2. Docker Images Pushed to Docker Hub

Showing crud-backend and crud-frontend tags

3. GitHub Actions — SUCCESS RUN

Full pipeline success execution

4. EC2 Instance Running Containers

Output of docker ps showing mongo, backend, frontend, and nginx

5. Working Application

Browser screenshot of http://13.203.213.222/

6. Nginx Reverse Proxy Config Check

Output of docker exec -it nginx cat /etc/nginx/conf.d/default.conf

7. API Test

Output of curl http://localhost:3000/api/tutorials
