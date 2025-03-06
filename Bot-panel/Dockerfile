# Stage 1: Build PHP (Laravel)
FROM php:8.2-fpm-alpine as php-builder

WORKDIR /var/www
COPY src/ /var/www/
COPY .env /var/www/.env

# Install composer
COPY --from=composer:2 /usr/bin/composer /usr/bin/composer

# Install PHP dependencies and prepare Laravel for production
RUN apk add --no-cache $PHPIZE_DEPS mysql-client mariadb-connector-c-dev \
    && docker-php-ext-install pdo_mysql \
    && composer install --no-dev --optimize-autoloader \
#    && php artisan key:generate \
 #   && php artisan config:cache \
    && php artisan route:cache \
    && php artisan view:cache

# Stage 2: Build frontend (Node + Vite)
FROM node:18-alpine as node-builder
WORKDIR /app
COPY src/ /app
RUN npm install
RUN npm run build

# Stage 3: Final image for PHP
FROM php:8.2-fpm-alpine as php-final

WORKDIR /var/www
RUN apk add --no-cache $PHPIZE_DEPS mysql-client mariadb-connector-c-dev ffmpeg \
    && docker-php-ext-install pdo_mysql \
    && pecl install redis \
    && docker-php-ext-enable redis \
    && apk del $PHPIZE_DEPS

# Copy backend from php-builder
COPY --from=php-builder /var/www /var/www

# Copy built frontend from node-builder
COPY --from=node-builder /app/public/build /var/www/public/build

RUN chown -R www-data:www-data storage bootstrap/cache && \
    chmod -R 775 storage bootstrap/cache

CMD ["php-fpm"]

# Stage 4: Final image for Nginx
FROM nginx:alpine as nginx-final
WORKDIR /var/www

# Copy our nginx configuration
COPY nginx.conf /etc/nginx/conf.d/default.conf

# Copy all application files (including static assets) from the final PHP image
COPY --from=php-final /var/www /var/www
