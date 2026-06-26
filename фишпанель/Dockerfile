FROM php:8.1-apache

# Install system dependencies for GD and other extensions
RUN apt-get update && apt-get install -y \
    libpng-dev \
    libjpeg-dev \
    libfreetype6-dev \
    && rm -rf /var/lib/apt/lists/*

# Install required PHP extensions
RUN docker-php-ext-configure gd --with-jpeg --with-freetype \
    && docker-php-ext-install -j$(nproc) pdo pdo_mysql gd

# Enable Apache mod_rewrite
RUN a2enmod rewrite

# >>> ИСПРАВЛЕНИЕ ОШИБКИ С MPM <<<
# Отключаем конфликтующие модули и принудительно включаем mpm_prefork
RUN a2dismod mpm_event mpm_worker || true
RUN a2enmod mpm_prefork

# Set ServerName to suppress warning
RUN echo "ServerName localhost" >> /etc/apache2/apache2.conf

# Allow .htaccess overrides in the web root
RUN sed -i 's|AllowOverride None|AllowOverride All|g' /etc/apache2/apache2.conf

# Copy application source into the web root
COPY . /var/www/html/

# Set proper ownership and permissions
RUN chown -R www-data:www-data /var/www/html \
    && find /var/www/html -type d -exec chmod 755 {} \; \
    && find /var/www/html -type f -exec chmod 644 {} \;

EXPOSE 80
