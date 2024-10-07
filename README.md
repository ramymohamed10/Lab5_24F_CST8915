# Lab 5 - CST8915 Full-stack Cloud-native Development: Containerizing the Algonquin Pet Store with Docker

Welcome to Lab 5 of the **CST8915 Full-stack Cloud-native Development** course. In this lab, you will learn how to containerize microservices using Docker and manage them with Docker Compose. The goal is to containerize the **Algonquin Pet Store** application, run it locally using Docker, and prepare it for cloud-based environments.


## Lab Objectives:
- Understand the fundamentals of Docker containers.
- Dockerize the `order-service`, `product-service`, and `store-front` microservices of the **Algonquin Pet Store**.
- Use **Docker Compose** to manage multi-container applications locally.
- Push Docker images to a container registry Docker Hub.

## Prerequisites:
- **Docker:** Install Docker Desktop on your local machine ([Windows](https://docs.docker.com/desktop/install/windows-install/)/[Mac](https://docs.docker.com/desktop/install/mac-install/)/[Linux](https://docs.docker.com/desktop/install/linux/)).

## Introduction to Docker and Docker Compose
### What is Docker?
Docker is an open-source platform that enables developers to automate the deployment of applications inside lightweight, portable containers. These containers bundle the application with all of its dependencies and configurations, ensuring that the application runs consistently across different environments.

Key Concepts of Docker:

- **Containers:** Containers are lightweight, standalone executable packages of software that include everything needed to run the application: code, runtime, system tools, libraries, and settings.
- **Images:** An image is a read-only template used to create Docker containers. It contains everything needed to run an application. When you run a container, Docker uses an image as the base.
- **Dockerfile:** A text file that contains instructions on how to build a Docker image. The Dockerfile defines what goes into the image, including the base operating system, dependencies, environment variables, and the application code.
- **Docker Hub:** A cloud-based registry where Docker images can be stored and shared. You can push images to Docker Hub and pull them from there on different systems or cloud platforms.

### What is Docker Compose?
Docker Compose is a tool that allows you to define and manage multi-container Docker applications. Instead of running multiple docker run commands to start each container, you can define all your services in a single docker-compose.yml file and start them with a single command.

Key Concepts of Docker Compose:

- **Service:** A service refers to a container in a Docker Compose configuration. Each service runs one image, but you can scale services to run multiple containers.
- **`docker-compose.yml`** file: This is the configuration file where you define all the services in your application. It specifies how the containers are built, connected, and run.
- **Networking:** Docker Compose automatically creates a network for all the containers defined in the `docker-compose.yml` file, allowing them to communicate with each other by service name.

## How Docker and Docker Compose Work Together
When you’re working with a project that consists of multiple microservices, you will typically use Docker to create images for each of the services. Once the images are created, you use Docker Compose to define how these images should run together, how they interact, and how they are networked.

For example:

- The `order-service`, `product-service`, and `store-front` will each be built into their own Docker images using Docker.
- **Docker Compose** will then be used to define how these containers (services) should run together, ensuring that the front-end (`store-front`) can communicate with the back-end services (`order-service` and `product-service`) and that services interact with RabbitMQ for messaging in needed.

## Lab Steps:

### Step 1: Fork and Clone the Repositories

To begin this lab, you need to **fork** the following repositories into your own GitHub account and then **clone** them to your local machine. This allows you to make changes and push code to **your own repositories**.


Here are the repositories that need to be forked:
- Order Service: https://github.com/ramymohamed10/order-service-L4
- Product Service (Python): https://github.com/ramymohamed10/product-service-python-L4
- Product Service (Rust): https://github.com/ramymohamed10/product-service-rust-L4
- Store Front: https://github.com/ramymohamed10/store-front-L4


#### Steps to Fork and Clone:
Fork the Repositories:

- Visit each repository link above.
- Click the "Fork" button in the top-right corner of the repository page to create a copy under your GitHub account.

Clone the Forked Repositories:
- Once the repositories are forked, go to your GitHub profile and navigate to the forked repository.
- Copy the clone URL for each repository.
- Open your terminal and clone the repositories to your local machine. Example:
  ```
  git clone https://github.com/<your-github-username>/order-service-L4.git
  git clone https://github.com/<your-github-username>/product-service-python-L4.git
  git clone https://github.com/<your-github-username>/product-service-rust-L4.git
  git clone https://github.com/<your-github-username>/store-front-L4.git
  ```



### Step 2: Containerizing the Algonquin Pet Store Microservices
In this step, you will create Dockerfiles for each of the `order-service`, `product-service`, and `store-front` microservices.

#### Step 2.1. Dockerize the order-service
In the `order-service` directory, create a `Dockerfile` with the following content:
  ```
  # Use an official Node.js runtime as a parent image
  FROM node:20-alpine

  # Set the working directory
  WORKDIR /usr/src/app

  # Copy package.json and package-lock.json
  COPY package*.json ./

  # Install dependencies
  RUN npm install 

  # Copy the rest of the application code
  COPY . .

  # Expose the service port
  EXPOSE 3000

  # Set environment variables for RabbitMQ and other configurations
  ENV RABBITMQ_CONNECTION_STRING=amqp://guest:guest@localhost:5672/
  ENV PORT=3000

  # Start the order-service
  CMD ["node", "index.js"]
  ```
**Docker file explanation:**
- **Base Image**: 
  - Uses the official Node.js runtime image, version `20-alpine` (a lightweight version of Node.js).  
- **Set Working Directory**: 
  - The working directory inside the container is set to `/usr/src/app`.
- **Copy Package Files**: 
  - Copies `package.json` and `package-lock.json` to the working directory.
- **Install Dependencies**: 
  - Runs `npm install` to install all dependencies specified in `package.json`.
- **Copy Application Code**: 
  - Copies the rest of the application code into the container.
- **Expose Service Port**: 
  - Exposes port `3000` for the application, making it accessible from that port.
- **Set Environment Variables**:
  - Defines environment variables for RabbitMQ connection and the service's port (`3000`).
- **Start the Service**:
  - Specifies the command to start the Node.js application, running `index.js` using Node.

  
**Build the Docker image:**
  ```
  docker build -t order-service:latest .
  ```

**Run a Docker container from order-service:latest image and expose it on port 3000:**
```
docker run --rm -d -p 3000:3000 order-service:latest
```

#### Step 2.2. Dockerize the product-service
You can choose between the **Python** or **Rust** implementation of the `product-service`.

**Python Version:** In the `product-service` directory, create a `Dockerfile` with the following content:
  ```
  # Use an official Python runtime as a parent image
  FROM python:3.10

  # Set the working directory
  WORKDIR /usr/src/app

  # Copy requirements.txt
  COPY requirements.txt .

  # Install dependencies
  RUN pip install --no-cache-dir -r requirements.txt

  # Copy the rest of the application code
  COPY . .

  # Expose the service port
  EXPOSE 3030

  # Set environment variables for the product-service
  ENV PORT=3030

  # Start the product-service
  CMD ["python", "app.py"]
  ```
**Docker file explanation:**
- **Base Image**: 
  - Uses the official Python runtime image, version `3.10`.
- **Set Working Directory**: 
  - The working directory inside the container is set to `/usr/src/app`.
- **Copy Requirements File**: 
  - Copies `requirements.txt` to the working directory.
- **Install Dependencies**: 
  - Runs `pip install --no-cache-dir -r requirements.txt` to install dependencies listed in `requirements.txt`.
- **Copy Application Code**: 
  - Copies the rest of the application code into the container.
- **Expose Service Port**: 
  - Exposes port `3030` for the application, making it accessible from that port.
- **Set Environment Variables**:
  - Defines the environment variable `PORT` as `3030` for the product-service.
- **Start the Service**:
  - Specifies the command to start the Python application, running `app.py` using Python.


**Rust Version:** In the `product-service` directory, create a `Dockerfile` with the following content:
  ```
  # Use the official Rust image to build the product-service
  FROM rust:1.70 as builder

  WORKDIR /usr/src/product-service

  # Copy the project files and dependencies
  COPY . .

  # Build the product-service
  RUN cargo build --release

  # Use a smaller image for running the service
  FROM debian:bullseye-slim

  WORKDIR /usr/src/app

  # Copy the compiled binary from the builder stage
  COPY --from=builder /usr/src/product-service/target/release/product-service /usr/local/bin/

  # Expose the service port
  EXPOSE 3030

  # Set environment variables for the product-service
  ENV PORT=3030

  # Run the product-service
  CMD ["product-service"]
  ```
**Docker file explanation:**
- **Base Image for Build**: 
  - Uses the official Rust image, version `1.70`, as a build environment (builder stage).
- **Set Working Directory**: 
  - The working directory for building is set to `/usr/src/product-service`.
- **Copy Project Files and Dependencies**: 
  - Copies the entire project (including dependencies) into the build environment.
- **Build the Product Service**: 
  - Runs `cargo build --release` to compile the Rust project in release mode.
- **Use Smaller Runtime Image**: 
  - Switches to a lightweight `debian:bullseye-slim` image for running the service.
- **Set Working Directory for Runtime**: 
  - The runtime working directory is set to `/usr/src/app`.
- **Copy Compiled Binary**: 
  - Copies the compiled binary from the build stage to the runtime environment's `/usr/local/bin/`.
- **Expose Service Port**: 
  - Exposes port `3030` for the service, making it accessible from that port.
- **Run the Product Service**: 
  - Specifies the command to run the compiled `product-service` binary.

**Build the Docker image for your chosen version:**
```
docker build -t product-service:latest .
```
**Run a Docker container from product-service:latest image and expose it on port 3030:**
```
docker run --rm -d -p 3030:3030 product-service:latest
```

#### How Multi-Stage Builds Minimize the Final Image Size

Multi-stage builds in Docker are a strategy to **minimize the size of the final image** by separating the build environment (which often includes many development tools and dependencies) from the runtime environment (which only needs the bare essentials to run the application).

#### 1. Separation of Build and Runtime Environments
- **First stage (builder):** 
  - The `rust:1.70` image is used, which includes the full Rust toolchain (`cargo`, `rustc`, and other libraries). This image is relatively large because it contains all the development tools necessary to compile a Rust project.
  - The project is compiled in **release mode** using `cargo build --release`, which creates a small, optimized binary of the Rust application. All intermediate artifacts (like object files, unused dependencies, and build tools) are not needed for the final image.
  
- **Second stage (runtime):**
  - After the application is built, the final image is created using a much smaller base image, **`debian:bullseye-slim`**. This image contains only the bare minimum needed to run a program (in this case, the Rust binary), but none of the tools required to build the program.
  - By not including the Rust compiler, Cargo, or development libraries in this stage, the final image becomes significantly smaller.

#### 2. Copying Only the Compiled Binary
- The compiled binary (`product-service`) from the `builder` stage is the only thing that is copied to the runtime image:

  ```dockerfile
  COPY --from=builder /usr/src/product-service/target/release/product-service /usr/local/bin/
  ```

#### Step 2.3. Dockerize the store-front
In the `store-front` directory, create a `Dockerfile`:
  ```
  # Build the Vue.js app
  FROM node:20 AS build-stage
  WORKDIR /app
  COPY package*.json ./
  RUN npm install
  COPY . .
  ENV VUE_APP_ORDER_SERVICE_URL=http://localhost:3000
  ENV VUE_APP_PRODUCT_SERVICE_URL=http://localhost:3030
  RUN npm run build

  # Nginx for serving the built app
  FROM nginx:alpine
  COPY --from=build-stage /app/dist /usr/share/nginx/html
  EXPOSE 80
  ```
**Docker file explanation:**
- **Base Image for Build**: 
  - Uses the official Node.js image, version `20`, as the build environment (build stage).

- **Set Working Directory**: 
  - The working directory for the build is set to `/app`.

- **Copy Package Files**: 
  - Copies `package.json` and `package-lock.json` to the working directory.

- **Install Dependencies**: 
  - Runs `npm install` to install the project dependencies.

- **Copy Application Code**: 
  - Copies the rest of the Vue.js application code into the build environment.

- **Build the Application**: 
  - Runs `npm run build` to compile the Vue.js application into production-ready static files.

- **Use Nginx to Serve the App**: 
  - Switches to the `nginx:alpine` image to serve the built application using Nginx.

- **Copy Built Files**: 
  - Copies the compiled files from the build stage (`/app/dist`) to Nginx's default serving directory (`/usr/share/nginx/html`).

- **Expose Service Port**: 
  - Exposes port `80` for the Nginx web server to serve the Vue.js app.


**Build the Docker image:**
```
docker build -t store-front:latest .
```
**Run a Docker container from store-front:latest image:**
```
docker run --rm -d -p 80:80 store-front:latest
```
  

### Step 3: Using Docker Compose for Local Development
Now that all microservices have been containerized, you will use Docker Compose to orchestrate the three services and run them together.



#### Step 3.1. Create a docker-compose.yml file
In the root directory of your project (at the same level as the `order-service`, `product-service`, and `store-front` directories), create a `docker-compose.yml` file:
  ```
# Specifies the version of Docker Compose format being used
version: '3'

services:
  # Definition for RabbitMQ service
  rabbitmq:
    # Uses the RabbitMQ Docker image with the management plugin enabled
    image: "rabbitmq:3-management"
    # Maps the container's internal RabbitMQ ports to the host machine
    ports:
      - "5672:5672"   # Default RabbitMQ port (for messaging)
      - "15672:15672" # Management plugin UI port (for monitoring and managing)
    # Environment variables for default user credentials for RabbitMQ
    environment:
      - RABBITMQ_DEFAULT_USER=myuser      # Sets the default RabbitMQ username
      - RABBITMQ_DEFAULT_PASS=mypassword  # Sets the default RabbitMQ password

  # Definition for Order Service
  order-service:
    # Builds the Docker image for order-service using the local directory
    build: ./order-service-L4
    # Maps the container's port 3000 to port 3000 on the host machine
    ports:
      - "3000:3000"
    # Environment variables for the Order Service
    environment:
      # Connection string for RabbitMQ, allowing communication with the messaging broker
      - RABBITMQ_CONNECTION_STRING=amqp://myuser:mypassword@rabbitmq:5672/
      - PORT=3000
    # Ensures that RabbitMQ starts before the Order Service
    depends_on:
      - rabbitmq

  # Definition for Product Service
  product-service:
    # Builds the Docker image for product-service using the local directory
    build: ./product-service-python-L4
    # Maps the container's port 3030 to port 3030 on the host machine
    ports:
      - "3030:3030"
    environment:
      # Connection string for RabbitMQ, allowing communication with the messaging broker
      - PORT=3030

  # Definition for Store Front (Frontend Service)
  store-front:
    # Builds the Docker image for store-front using the local directory
    build: ./store-front-L4
    # Maps the container's port 80 (HTTP) to port 80 on the host machine
    ports:
      - "80:80"
    # Environment variables for the Store Front Service
    environment:
      # URL to access the Order Service from within the Docker network
      - VUE_APP_ORDER_SERVICE_URL=http://order-service:3000
      # URL to access the Product Service from within the Docker network
      - VUE_APP_PRODUCT_SERVICE_URL=http://product-service:3030
    depends_on:
      - product-service
      - order-service
  ```

#### Step 3.2. Run the application using Docker Compose

In your terminal, run:
```
docker-compose up --build
```
This will build and run all the services together. You should be able to access the store-front at http://localhost:80.


### Step 4: Pushing Docker Images to a Container Registry
Push the Docker images to a container registry. You can use Docker Hub or Azure Container Registry (ACR). Here we are going to use Docker Hub.

You need to log in to your Docker Hub account before pushing the images:
```bash
docker login
```

#### Step 4.1 Tag the images:
  ```
  docker tag order-service:latest <your-dockerhub-username>/order-service:latest
  docker tag product-service:latest <your-dockerhub-username>/product-service:latest
  docker tag store-front:latest <your-dockerhub-username>/store-front:latest
  ```

#### Step 4.2 Push the images:
  ```
  docker push <your-dockerhub-username>/order-service:latest
  docker push <your-dockerhub-username>/product-service:latest
  docker push <your-dockerhub-username>/store-front:latest
  ```


### Optional: Clean Up Docker Environment
After completing the lab, you can clean up your local Docker environment to free up space by removing unused images and containers:

```bash
docker system prune -a
```