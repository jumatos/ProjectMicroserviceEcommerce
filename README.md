# ProjectMicroserviceEcommerce
__Ecommerce Web Project carried out in microservices and deployed in containers.__


![EMartApp](https://github.com/user-attachments/assets/2d8ad30a-2c7e-433d-bc1e-7ce7d550a989)

# Client Dockerfile
![image](https://github.com/user-attachments/assets/b0ac308a-1c84-468d-a4d1-670fb49794c1)

- __First stage (web-build):__ Builds the web client using Node.js and npm.
- __Second stage (nginx):__ Uses Nginx to serve the production build of the web client.
- __Nginx Configuration:__  A custom configuration file is used to control how Nginx serves the app.
- The final image will contain an Nginx server that serves the static files built by the Node.js application.

# Javaapi Dockerfile
![image](https://github.com/user-attachments/assets/1715153a-48da-4797-a610-c29fdd3fb2cc)

__First stage (BUILD_IMAGE):__
- Builds the Java application using Maven.
- Skips tests during the build process for faster builds.
  
__Second stage:__

- Uses a clean openjdk:8 image to run the application (no Maven needed here).
- Copies the built .jar file from the first stage.
- Exposes port 9000 and sets the entry point to run the application using the Java runtime.
  
This approach separates the build and runtime environments, leading to a smaller and more efficient final image, since Maven and build dependencies are not included in the runtime image.

# Nodeapi Dockerfile
![image](https://github.com/user-attachments/assets/cb472a59-84d5-4554-bff7-d21646215b60)

__First stage (nodeapi-build):__
- Builds the Node.js application by installing dependencies (npm install).
  
__Second stage:__

- Sets up the environment to run the application.
- Copies the built application and installed dependencies from the first stage.
- Exposes port 5000 for the application.
- Defines the command to start the application (npm start).
  
__Benefits of Using Multi-Stage Build:__

- Cleaner final image: The second stage contains only the necessary files to run the app, without the build tools (like npm) that are only needed during the build process.
- Reduced image size: Since the dependencies are installed in the first stage and the build tools are not included in the final image, the resulting image will be smaller and more efficient for deployment.
- 
# Multi-stage Docker images  (Dockerfile)
![image](https://github.com/user-attachments/assets/0c7dd4b0-ed3e-4a45-99c9-0a29d9bd26a1)

This Dockerfile is designed to build and package a multi-stage Docker image for a full-stack application. The application consists of a frontend (in client/, built with Angular) and a backend (in nodeapi/, built with Node.js). The image is optimized using multi-stage builds to reduce the final image size.

__ui-build:__

- Builds the Angular frontend and creates the dist/ folder.

__server-build:__

- Installs dependencies for the Node.js backend.

__Final Image:__

- Copies the built frontend and backend into the final image.
- Exposes the required ports (4200 for frontend, 5000 for backend).
- Runs the application with npm start.
  
__Key Points__

- __Multi-Stage Builds:__ The Dockerfile uses multi-stage builds to keep the final image lean. It first builds the frontend and backend separately, and then in the final stage, it combines them into one image.

- __Efficiency:__ By separating the build steps for the frontend and backend, Docker caches each build stage. This means that only the stages that change (e.g., code changes) will need to be rebuilt, improving build efficiency.

- __Port Exposing:__ Ports 4200 (frontend) and 5000 (backend) are exposed to allow external access to the services.

This approach is common in modern full-stack Node.js applications where the frontend and backend are combined into a single Docker image for easier deployment, particularly in development environments.

# Docker-Compose.yaml

This code is a Docker Compose configuration file that defines a multi-container environment for an application. The application is based on a microservices architecture, with separate services for the frontend, backend, database, and reverse proxy (Nginx).

The version 3.8 indicates it's using Docker Compose syntax for version 3.8, which is suitable for production use with some advanced features.

Let's go through each service defined in the docker-compose.yml file:

__1. client (Frontend Service)__

![image](https://github.com/user-attachments/assets/af8cb883-0e7b-4ad8-bafa-95f82ee8bb6f)

- __build: context: ./client:__ Builds the client image from the files in the ./client directory (the frontend code).
- __ports: - "4200:4200":__ Exposes port 4200 of the container to port 4200 on the host machine, which is the default port for Angular applications.
- __container_name: client:__ Names the container client.
- __depends_on: - api - webapi:__ This specifies that the client service depends on the api and webapi services. The client will wait for these services to be available before starting.


__2. api (Node.js API Service)__

![image](https://github.com/user-attachments/assets/471ad945-54ab-4426-a6b5-659515a542f8)

- __build: context: ./nodeapi:__ Builds the api image from the ./nodeapi directory, which contains the backend code written in Node.js (typically using Express or a similar framework).
- __ports: - "5000:5000":__ Exposes port 5000 of the container to port 5000 on the host machine.
- __restart: always:__ Ensures that the api service always restarts if it crashes.
- __container_name: api:__ Names the container api.
- __depends_on: - nginx - emongo:__ Specifies that api depends on the nginx and emongo services. It ensures that the API will only start once nginx (the reverse proxy) and emongo (the MongoDB database) are up.

__3. webapi (Java API Service)__

![image](https://github.com/user-attachments/assets/809ab6a6-bd74-416b-8dfb-79fa4ef92eea)

- __build: context: ./javaapi:__ Builds the webapi image from the ./javaapi directory, which contains the backend code for a Java-based API (likely using Spring Boot or similar).
- __ports: - "9000:9000":__ Exposes port 9000 of the container to port 9000 on the host machine.
- __restart: always:__ Ensures the webapi service restarts automatically if it crashes.
- __container_name: webapi:__ Names the container webapi.
- __depends_on: - emartdb:__ The webapi depends on emartdb, which is the MySQL database. It ensures that webapi will only start once emartdb is up.

__4. nginx (Reverse Proxy)__

![image](https://github.com/user-attachments/assets/757d8265-8d30-4c34-85f0-f7f94a7e1c18)

__image: nginx:latest:__ Uses the official Nginx image from Docker Hub (nginx:latest) to act as a reverse proxy.

__container_name: nginx:__ Names the container nginx.

__volumes: - "./nginx/default.conf:/etc/nginx/conf.d/default.conf":__ Mounts a custom Nginx configuration file (./nginx/default.conf) from the host to the container. This file likely contains settings for routing traffic to the client, api, and webapi services.

__ports: - "80:80":__ Exposes port 80 of the container (the default HTTP port) to port 80 on the host, allowing access to the services through Nginx.

__5. emongo (MongoDB Database)__

![image](https://github.com/user-attachments/assets/3221b5b6-46cb-4ced-9fa6-167e78338820)

- __image: mongo:4:__ Uses the official MongoDB image version 4 from Docker Hub.
- __container_name: emongo:__ Names the container emongo.
- __environment: - MONGO_INITDB_DATABASE=epoc:__ Initializes the MongoDB database with the name epoc.
- __ports: - "27017:27017":__ Exposes port 27017 of the container (the default MongoDB port) to port 27017 on the host machine.

__6. emartdb (MySQL Database)__

![image](https://github.com/user-attachments/assets/26f78aa5-1d99-4ca0-9d1e-00db53a9eb97)

- __image: mysql:8.0.33:__ Uses the official MySQL image version 8.0.33.
- __container_name: emartdb:__ Names the container emartdb.
- __ports: - "3306:3306":__ Exposes port 3306 of the container (the default MySQL port) to port 3306 on the host machine.
  
__environment: - MYSQL_ROOT_PASSWORD=emartdbpass - MYSQL_DATABASE=books:__ Sets environment variables to configure the MySQL database:

- __MYSQL_ROOT_PASSWORD:__ The root password for the MySQL database (emartdbpass).
- __MYSQL_DATABASE:__ The name of the initial database to create (books).

# How it Works

- 1. Client runs the frontend (Angular) and connects to the backend services (api and webapi).
- 2. API (api) is a Node.js-based service that communicates with MongoDB (emongo).
- 3. Web API (webapi) is a Java-based service (likely Spring Boot) that interacts with MySQL (emartdb).
- 4. Nginx acts as a reverse proxy, routing incoming HTTP requests to the appropriate service.
- 5. MongoDB and MySQL are the databases for the application, with MongoDB used by the Node.js backend and MySQL used by the Java backend.

# Step-by-Step Deployment:
We create a virtual machine in EC2 with:
- Operating System: Ubuntu.
- Capacity: t3.medium.
- Hard disk space: 20gb.
![Screenshot from 2025-03-05 10-06-06](https://github.com/user-attachments/assets/76903e25-efe5-47ee-a501-484bd56c376b)
![Screenshot from 2025-03-05 10-05-33](https://github.com/user-attachments/assets/f0b866f6-a955-4911-84bc-e67d20b79fe4)

- We place in "User data" the script to install docker and docker-compose.
![Screenshot from 2025-03-05 09-17-15](https://github.com/user-attachments/assets/86bd6e84-5eb7-49e4-a842-3225c8b5bb4c)

We perform a __git clone__ and then a __docker-compose build__:
![Screenshot from 2025-03-05 09-16-04](https://github.com/user-attachments/assets/d53dd0cd-f05d-4640-8c59-5bf6961a2a2b)

 __docker-compose up__:
![Screenshot from 2025-03-05 10-56-25](https://github.com/user-attachments/assets/821d320b-514d-4d47-a418-d01e12c5c1e4)
![Screenshot from 2025-03-05 11-03-53](https://github.com/user-attachments/assets/e4270542-e920-47b8-af72-3fbae18885ed)

__Ready, the application is now running in containers:__
![Screenshot from 2025-03-05 10-58-03](https://github.com/user-attachments/assets/b250cecf-2e32-4fd1-a265-264dc21cfa6c)
![Screenshot from 2025-03-05 10-58-36](https://github.com/user-attachments/assets/6badcfba-b51b-45b2-b2b4-76506aac2c80)
![Screenshot from 2025-03-05 11-03-12](https://github.com/user-attachments/assets/41c58ed1-df02-44f1-ad45-ff4950903cd3)
![Screenshot from 2025-03-05 11-00-24](https://github.com/user-attachments/assets/006ca025-3634-49aa-9c20-c8fca28c9ddd)
![Screenshot from 2025-03-05 11-00-11](https://github.com/user-attachments/assets/cf762c4c-4c6c-49df-bdeb-8e4aa03bea8f)



