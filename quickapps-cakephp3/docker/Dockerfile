FROM php:7.4-apache

# Install system dependencies
RUN apt-get update && apt-get install -y \
    git \
    curl \
    libpng-dev \
    libonig-dev \
    libxml2-dev \
    libicu-dev \
    zip \
    unzip \
    libzip-dev \
    libfreetype6-dev \
    libjpeg62-turbo-dev \
    libmcrypt-dev \
    default-mysql-client \
    && rm -rf /var/lib/apt/lists/*

# Install PHP extensions required by QuickAppsCMS
RUN docker-php-ext-configure gd --with-freetype --with-jpeg \
    && docker-php-ext-install -j$(nproc) \
    pdo \
    pdo_mysql \
    mbstring \
    exif \
    pcntl \
    bcmath \
    gd \
    intl \
    zip \
    opcache \
    fileinfo

# Install mcrypt extension via PECL (required for CakePHP 3/QuickAppsCMS)
RUN pecl install mcrypt-1.0.4 \
    && docker-php-ext-enable mcrypt

# Enable Apache mod_rewrite
RUN a2enmod rewrite

# Install Composer
COPY --from=composer:2 /usr/bin/composer /usr/bin/composer

# Set working directory
WORKDIR /var/www/html

# Configure Apache
COPY apache-config.conf /etc/apache2/sites-available/000-default.conf

# Set proper permissions
RUN chown -R www-data:www-data /var/www/html

# Ensure writable directories for QuickAppsCMS
RUN mkdir -p /var/www/html/tmp/cache/models \
    /var/www/html/tmp/cache/persistent \
    /var/www/html/tmp/cache/views \
    /var/www/html/tmp/sessions \
    /var/www/html/logs \
    && chmod -R 777 /var/www/html/tmp \
    && chmod -R 777 /var/www/html/logs

# PHP Configuration
RUN echo "memory_limit = 256M" >> /usr/local/etc/php/conf.d/custom.ini \
    && echo "post_max_size = 64M" >> /usr/local/etc/php/conf.d/custom.ini \
    && echo "upload_max_filesize = 64M" >> /usr/local/etc/php/conf.d/custom.ini \
    && echo "max_execution_time = 300" >> /usr/local/etc/php/conf.d/custom.ini

EXPOSE 80

CMD ["apache2-foreground"]