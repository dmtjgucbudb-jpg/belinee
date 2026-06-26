FROM php:8.1-apache

# Установка системных зависимостей для капчи (GD)
RUN apt-get update && apt-get install -y \
    libpng-dev \
    libjpeg-dev \
    libfreetype6-dev \
    && rm -rf /var/lib/apt/lists/*

# Установка расширений PHP
RUN docker-php-ext-configure gd --with-jpeg --with-freetype \
    && docker-php-ext-install -j$(nproc) pdo pdo_mysql gd

# Включаем обычный rewrite для ссылок
RUN a2enmod rewrite

# Настройки Apache
RUN echo "ServerName localhost" >> /etc/apache2/apache2.conf
RUN sed -i 's|AllowOverride None|AllowOverride All|g' /etc/apache2/apache2.conf

# Копируем проект внутрь сервера
COPY . /var/www/html/

# Выставляем права на файлы
RUN chown -R www-data:www-data /var/www/html \
    && find /var/www/html -type d -exec chmod 755 {} \; \
    && find /var/www/html -type f -exec chmod 644 {} \;

# Права на сессии
RUN mkdir -p /var/lib/php/sessions && chmod -R 777 /var/lib/php/sessions || true

# Настройка портов под Railway
RUN sed -i 's/Listen 80/Listen ${PORT}/g' /etc/apache2/ports.conf
RUN sed -i 's/<VirtualHost \*:80>/<VirtualHost \*:${PORT}>/g' /etc/apache2/sites-available/000-default.conf

CMD bash -c "rm -f /etc/apache2/mods-enabled/mpm_event.* /etc/apache2/mods-enabled/mpm_worker.* && apache2-foreground"
