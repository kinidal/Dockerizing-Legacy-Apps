## ![Web App](https://img.shields.io/badge/Web%20App-0078D7?style=for-the-badge&logo=googlechrome&logoColor=white) **The Angular App: Frontend**

### ✅ **Understanding the How the app is hosted in local dev environment**

As per the discussion with the developer they have forwarded the steps they perform for building and running the app locally.

1. Checkout the code from the git
2. Go to a location in the checkout code myapp_frontend/myapp/   
3. run # npm i to install the dependency packages
4. run npm start
5. Then the application will be hosted in a port 4200

### ✅ **Creating the Docker file**

**Docker File***
```plaintext
# Use official Node.js image as base
FROM node:18

# Set working directory inside the container
WORKDIR /app

# Copy package.json and package-lock.json first (to improve caching)
COPY package*.json ./

# Install dependencies
RUN npm install

# Copy the rest of the application files
COPY . .

# Expose the Angular dev server port
EXPOSE 4200

# Start the Angular application
CMD ["npm", "start"]

````

### ✅ **Setting UP the containers **

**Steps**
````Plaintext
docker build -t myyapp-frontend .

docker run -d -p 4200:4200 --name myyapp-frontend myapp-frontend

````

## 🔥 **Challenges Faced & How I Solved Them**

### ❌ Issue 1: The container is crashing showing npm related issues

> **My first Docker file was like this**  

```plaintext
# Use official Node.js image as base
FROM node:18

# Set working directory inside the container
WORKDIR /app

# Copy the application code (or use a bind mount for local development)
COPY . .

# Install dependencies
RUN npm install

# Expose port 4200
EXPOSE 4200

# Start the application
CMD ["npm", "start"]

````

> I have checked the logs using 'docker logs' and found that some issues related to npm start
> I checked if any other services using the same port 4200, none i have seen.
> Then i tried executing npm start using interactive mode

````plaintext
docker run -it --rm myyapp /bin/bash

npm install
npm start

curl http://localhost:4200
````
> This confirms that the application is working inside the container after executing the npm start command directly inside the container.
> After searching i found out that the Angular applications has these sort of issues when there is some dependencies in the files it fails to start the app. So i decided to copy the whole files after installing all the dependencies.
> After that container is UP issue is resolved

### ❌ Issue 2: The container is UP but the app is not available outside but inside

> The app is now running inside the container without issue but the same is not available in the outside local network
> After researching i found that there is a default setting in Angular apps that the default binding defined is to local system only
> I changed the package.json file as follows and it worked perfectly
````plaintext
"start": "ng serve --host 0.0.0.0 --port 4200 --disable-host-check"
````

---
