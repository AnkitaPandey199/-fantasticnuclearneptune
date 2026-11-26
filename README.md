# Phase 1: Containerization

1. Initialize Astro App
# Astro Starter Kit: Basics

```sh
npm create astro@latest -- --template basics
```

> 🧑‍🚀 **Seasoned astronaut?** Delete this file. Have fun!

## 🚀 Project Structure

Inside of your Astro project, you'll see the following folders and files:

```text
/
├── public/
│   └── favicon.svg
├── src
│   ├── assets
│   │   └── astro.svg
│   ├── components
│   │   └── Welcome.astro
│   ├── layouts
│   │   └── Layout.astro
│   └── pages
│       └── index.astro
└── package.json
```

To learn more about the folder structure of an Astro project, refer to [our guide on project structure](https://docs.astro.build/en/basics/project-structure/).

## 🧞 Commands

All commands are run from the root of the project, from a terminal:

| Command                   | Action                                           |
| :------------------------ | :----------------------------------------------- |
| `npm install`             | Installs dependencies                            |
| `npm run dev`             | Starts local dev server at `localhost:4321`      |
| `npm run build`           | Build your production site to `./dist/`          |
| `npm run preview`         | Preview your build locally, before deploying     |
| `npm run astro ...`       | Run CLI commands like `astro add`, `astro check` |
| `npm run astro -- --help` | Get help using the Astro CLI                     |

## 👀 Want to learn more?

Feel free to check [our documentation](https://docs.astro.build) or jump into our [Discord server](https://astro.build/chat).

# 2. Multi-Stage Dockerfile

Create a Dockerfile in the root directory to ensurea lightweight production image.
---------------------------------------------------
# Stage1: Build
FROM node:22-alpine AS build

WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build


# Stage2: Serve
FROM nginx:alpine
COPY --from=build /app/dist /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]

-----------------------------------------------

# 3.Build and test locally
Build the docker image and run it locally to verify functionality.

------------------------------------------------
docker build -t astro-app:latest .
docker run -d -p 8080:80 astro-app:latest
------------------------------------------------

Access http://localhost:8080 in your browser to confirm the app is running.

# Phase 2: Local Kubernetes Orchestration (Minikube)

Ensure Minikube and kubectl are installed.Start your local cluster from terminal:

minikube  start

Create a folder K8s in the project folder:

# 1. Kubernetes Deployment(k8s/deployment.yaml)

This file defines the desired state for your application pods.

----------------------------------------------------
apiVersion: apps/v1
kind: Deployment
metadata:
  name: astro-art-app-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: astro-art-app
  template:
    metadata:
      labels:
        app: astro-art-app
    spec:
      containers:
        - name: astro-art-app
          image: ghcr.io/ankitapandey199(your-github-name)/astro-art-app:latest
          ports:
            - containerPort: 80
      imagePullSecrets:
        - name: ghcr-secret

---------------------------------------------------

# 2. Kubernetes Service(k8s/service.yaml)

This exposes your deployment to traffic.

--------------------------------------------------

apiVersion: v1
kind: Service
metadata:
  name: astro-art-app
spec:
  type: LoadBalancer
  selector:
    app: astro-art-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
-------------------------------------------

# 3.Deploy to Minikube
Apply the configurations to your local cluster.

-------------------------------------------
# Point terminal to Minikube's docker daemon to use local images
eval $(minikube docker-env)

# Rebuild image inside minikube environment
docker build -t astro-app:latest .

# Apply manifests
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml

# Access the service
minikube tunnel
minikube service astro-app-service

---------------------------------------------

# Phase 3: Automated CI/CD with GitHub Actions , save and Push the changes.

 Workflow Configuration - Create a folder in project root directory (.github\workflows\docker-build.yml)

 --------------------------------------------

 name: Deploy Docker Image
on:
  push:
    branches:
      - main
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Log in to GITHUB Container Registry
        uses: docker/login-action@v1
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.CR_PAT }} #add github Personal access token (classic) , add in repo settings/secrets/actions

      - name: Build and push Docker image
        uses: docker/build-push-action@v2
        with:
          context: .
          push: true
          tags: ghcr.io/ankitapandey199(your-github-name)/astro-art-app:latest
-------------------------------------------------------








