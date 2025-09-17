FROM php:8.2-apache

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
    libsqlite3-dev \
    && rm -rf /var/lib/apt/lists/*

# Install PHP extensions required by CakePHP 5
RUN docker-php-ext-configure gd --with-freetype --with-jpeg \
    && docker-php-ext-install -j$(nproc) \
    pdo \
    pdo_mysql \
    pdo_sqlite \
    mbstring \
    exif \
    pcntl \
    bcmath \
    gd \
    intl \
    zip \
    opcache \
    fileinfo \
    xml

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

# PHP Configuration for CakePHP 5
RUN echo "memory_limit = 512M" >> /usr/local/etc/php/conf.d/custom.ini \
    && echo "post_max_size = 128M" >> /usr/local/etc/php/conf.d/custom.ini \
    && echo "upload_max_filesize = 128M" >> /usr/local/etc/php/conf.d/custom.ini \
    && echo "max_execution_time = 300" >> /usr/local/etc/php/conf.d/custom.ini \
    && echo "date.timezone = UTC" >> /usr/local/etc/php/conf.d/custom.ini

EXPOSE 80

CMD ["apache2-foreground"]