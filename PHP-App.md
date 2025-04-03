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

### ✅ **Setting UP the containers**

**Steps**
````Plaintext
docker build -t my-php55-app .
docker run -d -p 8080:80 my-php55-app
````

### ✅ **Setting UP the DataBase**


## 🔥 **Challenges Faced & How I Solved Them**

### ❌ Issue 1: Build image error : Base image and dependencies not found


