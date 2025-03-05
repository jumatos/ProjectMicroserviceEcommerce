# ProjectMicroserviceEcommerce
Ecommerce Web Project carried out in microservices and deployed in containers.


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
