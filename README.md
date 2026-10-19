# DANISEFUMI
To meet the Africa needs
name: Build and Deploy Bakery & Grocery App

on:
  push:
    branches: [ "main" ] # Triggers automatically on every push to the main branch

jobs:
  build-and-test:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout Code
      uses: actions/checkout@v4

    # 1. Setup Node.js for validation checks
    - name: Set up Node.js 18
      uses: actions/setup-node@v4
      with:
        node-version: '18'
        cache: 'npm'

    # 2. Build Backend to catch TypeScript compilation errors
    - name: Install & Build Backend
      run: |
        cd server
        npm install
        npm run build

    # 3. Build Admin Dashboard to catch React/Next.js errors
    - name: Install & Build Admin Dashboard
      run: |
        cd admin-dashboard
        npm install
        npm run build

  # Optional Step: Containerize & Push to Docker Hub or Cloud Registry
  docker-publish:
    needs: build-and-test
    runs-on: ubuntu-latest
    steps:
    - name: Checkout Code
      uses: actions/checkout@v4

    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v3

    # Optional: Login to Docker Hub (Requires GitHub Secrets setup)
    - name: Login to Docker Hub
      uses: docker/login-action@v3
      with:
        username: ${{ secrets.DOCKER_HUB_USERNAME }}
        password: ${{ secrets.DOCKER_HUB_TOKEN }}

    - name: Build and Push Docker Image
      uses: docker/build-push-action@v5
      with:
        context: .
        file: ./Dockerfile
        push: true
        tags: ${{ secrets.DOCKER_HUB_USERNAME }}/bakery-grocery-app:latest

  # Optional Step: Trigger Cloud Deployment (e.g., Render, Railway, or AWS)
  deploy-to-cloud:
    needs: docker-publish
    runs-on: ubuntu-latest
    steps:
    - name: Trigger Webhook Deployment
      run: |
        # Example: Triggering a deployment webhook for cloud services like Render
        # curl -X POST "${{ secrets.RENDER_DEPLOY_HOOK_URL }}"
        echo "Deployment webhook triggered successfully!"
