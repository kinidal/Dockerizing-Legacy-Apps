# Dockerizing Legacy Apps Managing via Azure Container Services.

![Docker](https://img.shields.io/badge/Docker-Containerization-blue?logo=docker)
![DevOps](https://img.shields.io/badge/DevOps-CI/CD-orange?logo=githubactions)
![Python](https://img.shields.io/badge/Python-Django-green?logo=python)
![Angular](https://img.shields.io/badge/Angular-Frontend-red?logo=angular)

## 📌 **Introduction**
This project demonstrates the **containerization** and **deployment** of a full-stack Legacy apps with underlying different technologies such as **Angular, Python, PHP and ASP.Net**.
It showcases my **DevOps** skills in setting up microservices, handling different tech stacks, debugging, and automating deployments.

### 🔹 **Tech Stack**
- **Angular**
- **Python and Django (REST API)**
- **PHP**
- **ASP.Net**
- **Database:MySQL**
- **Docker**
- **Azure Container Services**

---

## 🎯 **Motive & Plan**

### ✅ **Motive**
- Gain hands-on experience in **Docker & DevOps**
- Automate **frontend & backend deployments** using containers
- Learn **troubleshooting** and **debugging** in a containerized environment
- Manage the containers, **Pull and Push via Azure container services**

### ✅ **Plan / Expected Outcome**
1. **Containerize** the Legacy **Angular, Python, PHP and ASP.Net** apps
2. **Deploy** the containerized apps
3. **Resolve common deployment issues**
4. Manage the containers using the images stores in **Azure Container Services**

---

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

## ![Web App](https://img.shields.io/badge/Web%20App-0078D7?style=for-the-badge&logo=googlechrome&logoColor=white) **The Angular App: Backend**

### ✅ **Understanding the How the app is hosted in local dev environment**

1. Checkout the code
2. going inside the app directory #cd myapp-backend
3. Create env
4. #pip install -r requirements.txt
5. python manage.py runserver

### ✅ **Creating the Docker file**

**DockerFile**
````plaintext
# Use official Python image as base
FROM python:3.10

# Set the working directory inside the container
WORKDIR /app

# Copy the application code to the container
COPY . .

# Install dependencies
RUN pip install --no-cache-dir -r requirements.txt

# Expose the port Django runs on
EXPOSE 8000

# Start the Django server
CMD ["python", "manage.py", "runserver", "0.0.0.0:8000"]

````

### ✅ **Setting UP the containers **

**Steps**
````Plaintext
docker build -t myapp-backend .
docker run -d -p 8000:8000 --name myappp-backend myapp-backend
````

## 🔥 **Challenges Faced & How I Solved Them**

### ❌ Issue 1: Image buil error : The Python version issue

When i am trying to build the image the build is throwing error in the pip install requirements.txt
In this requirements.txt  files all the dependencies are mentioned, like django,...etc
When i checked with the development team they have informed the current version of the python is not supporting some of the libraries added in the requirements.txt file
So I changed the base image from latest to Python 3.10 which resolved the issue and am able to successfully create the image.

### ❌ Issue 2: The container is crashing due to Environment variable file missing

I have checked the logs of the container using **docker logs myapp-backend** command and found that this is an issue due to .env file is not present in the code files of the container, since the environment variables are set using this .env file its absence is creating the issue. I then copied the .env file to the container (during image build itself) and re run the container which resolved the issue.


---

## ![Web App](https://img.shields.io/badge/Web%20App-0078D7?style=for-the-badge&logo=googlechrome&logoColor=white) **The PHP Application**

I have a Legacy PHP application which works in old deprecated version > PHP version 5.5.38. Due to some dependencies the version can't be upgraded.


### ✅ **Creating the Docker file**

**DockerFile**
````plaintext
# Use the base image with PHP 5.5 
FROM crazymax/php-old:5.5-apache

# Set maintainer info
LABEL maintainer="nidalki@gmail.com"

# Update package lists and install required dependencies
RUN apt-get update && apt-get install -y --no-install-recommends --allow-unauthenticated \
    libfreetype6-dev \
    libjpeg62-turbo-dev \
    libpng12-dev \
    libxml2-dev \
    unzip \
    zlib1g-dev \
    && docker-php-ext-install -j$(nproc) iconv \
    && docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
    && docker-php-ext-install -j$(nproc) gd \
    && docker-php-ext-install pdo pdo_mysql mysqli soap \
    && rm -rf /var/lib/apt/lists/*  # Cleanup to reduce image size

# Enable Apache rewrite module
RUN a2enmod rewrite

# Set the working directory
WORKDIR /var/www/html

# Copy application files
COPY . /var/www/html

# Set proper permissions
RUN chown -R www-data:www-data /var/www/html

# Expose the required port
EXPOSE 80

# Start Apache
CMD ["apache2-foreground"]

````

### ✅ **Creating the PHP.ini file**

**php.ini**
````
; General settings
memory_limit = 256M
upload_max_filesize = 20M
post_max_size = 20M
max_execution_time = 300
date.timezone = UTC

; Error handling
display_errors = Off
display_startup_errors = Off
error_reporting = E_ALL & ~E_DEPRECATED & ~E_STRICT
log_errors = On
error_log = /var/log/php_errors.log

; Security
expose_php = Off
session.cookie_httponly = 1
````

### ✅ **Setting UP the containers **

**Steps**
````Plaintext
docker build -t my-php55-app .
docker run -d -p 8080:80 my-php55-app
````

### ✅ **Setting UP the DataBase **


## 🔥 **Challenges Faced & How I Solved Them**

### ❌ Issue 1: Build image error : Base image and dependencies not found


## ![Web App](https://img.shields.io/badge/Web%20App-0078D7?style=for-the-badge&logo=googlechrome&logoColor=white) **The ASP.Net Application**

I have a Legacy ASP.Net application which works in old deprecated version 3.5 .Net Framework.
Due to some dependencies the version can't be upgraded. Since the application is a .Net framework application am planning to use a windows system as host for the docker and run windows containers in it.

Also my plan is to create ***Multi Stage*** Build so as to reduce the final image file size and improved security.

Also while discussing with the developers they told that there is no separate build steps they were performing when they are locally setting up the applciation. Instead they do like **open the .sln file drectly in to Visual studio Pro** and run the app in to the IIS using the button in the VS pro itself.

Hence finding the build steps and knowing the build files locations was bit tricky.


### ✅ **Creating the Docker file**
**DockerFile**

````plaintext
# Build stage
FROM mcr.microsoft.com/dotnet/framework/sdk:3.5-windowsservercore-ltsc2019 AS build
WORKDIR /app

# Copy solution and project files
COPY ./source\ code/PMOscar.sln .
COPY ./source\ code/ .

# Build the application
RUN msbuild PMOscar.sln /p:Configuration=Release

# Final stage
FROM mcr.microsoft.com/dotnet/framework/aspnet:3.5-windowsservercore-ltsc2019
WORKDIR /inetpub/wwwroot

# Copy ALL web content files from the source
COPY --from=build /app/ .

# Configure IIS using appcmd instead of PowerShell cmdlets
RUN powershell -Command \
    Import-Module WebAdministration; \
    Remove-WebSite -Name 'Default Web Site'; \
    New-WebSite -Name 'PMOscar' -Port 80 -PhysicalPath 'C:\inetpub\wwwroot'; \
    New-WebAppPool -Name 'PMOscarPool'; \
    Set-ItemProperty 'IIS:\AppPools\PMOscarPool' -Name managedRuntimeVersion -Value 'v2.0'; \
    Set-ItemProperty 'IIS:\AppPools\PMOscarPool' -Name enable32BitAppOnWin64 -Value $true; \
    & $env:windir\system32\inetsrv\appcmd.exe set app 'PMOscar/' /applicationPool:'PMOscarPool'

# Expose port
EXPOSE 80

# Start IIS
ENTRYPOINT ["C:\\ServiceMonitor.exe", "w3svc"]
````

### ✅ **Setting UP the containers **

**Steps**
````Plaintext

docker build -t pmoscar-app .
docker run -d -p 4963:80 --name pmoscar pmoscar-app

````

## 🔥 **Challenges Faced & How I Solved Them**

### ❌ Issue 1: Build image error : Base image and dependencies not found
