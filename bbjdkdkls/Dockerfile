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

# Включаем модуль rewrite
RUN a2enmod rewrite

# Set ServerName to suppress warning
RUN echo "ServerName localhost" >> /etc/apache2/apache2.conf

# Allow .htaccess overrides in the web root
RUN sed -i 's|AllowOverride None|AllowOverride All|g' /etc/apache2/apache2.conf

# Copy application source into the web root
COPY . /var/www/html/

# >>> ЖЁСТКИЙ ХАК ДЛЯ РУССКОЙ ПАПКИ <<<
# Этот скрипт автоматически пропишет правила для Apache, чтобы он понимал кириллицу в URL
RUN echo 'RewriteEngine On' > /var/www/html/.htaccess && \
    echo 'RewriteCond %{REQUEST_URI} ^/%D1%84%D0%B0%D1%81%D0%B0%D0%B4(.*)$ [NC,OR]' >> /var/www/html/.htaccess && \
    echo 'RewriteCond %{REQUEST_URI} ^/%D1%84%D0%B8%D1%88%D0%BF%D0%B0%D0%BD%D0%B5%D0%BB%D1%8C(.*)$ [NC]' >> /var/www/html/.htaccess && \
    echo 'RewriteRule ^[^/]+/(.*)$ /var/www/html/фишпанель/$1 [L]' >> /var/www/html/.htaccess

# Set proper ownership and permissions
RUN chown -R www-data:www-data /var/www/html \
    && find /var/www/html -type d -exec chmod 755 {} \; \
    && find /var/www/html -type f -exec chmod 644 {} \;

# Выдаем права на папки сессий PHP
RUN mkdir -p /var/lib/php/sessions && chmod -R 777 /var/lib/php/sessions || true

# Заставляем Apache слушать порт, который выдает Railway
RUN sed -i 's/Listen 80/Listen ${PORT}/g' /etc/apache2/ports.conf
RUN sed -i 's/<VirtualHost \*:80>/<VirtualHost \*:${PORT}>/g' /etc/apache2/sites-available/000-default.conf

# Окончательная очистка модулей при старте
CMD bash -c "rm -f /etc/apache2/mods-enabled/mpm_event.* /etc/apache2/mods-enabled/mpm_worker.* && apache2-foreground"
