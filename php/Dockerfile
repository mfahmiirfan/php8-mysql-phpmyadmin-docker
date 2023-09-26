FROM php:8.0.27-apache
ADD https://github.com/mlocati/docker-php-extension-installer/releases/latest/download/install-php-extensions /usr/local/bin/
RUN chmod +x /usr/local/bin/install-php-extensions && \
    install-php-extensions mysqli intl zip

# install sqlsrv
RUN curl https://packages.microsoft.com/keys/microsoft.asc | tee /etc/apt/trusted.gpg.d/microsoft.asc

#Download appropriate package for the OS version
#Choose only ONE of the following, corresponding to your OS version

#Debian 11
RUN curl https://packages.microsoft.com/config/ubuntu/22.04/prod.list | tee /etc/apt/sources.list.d/mssql-release.list

RUN apt-get update
RUN ACCEPT_EULA=Y apt-get install -y msodbcsql17
# optional: for bcp and sqlcmd
RUN ACCEPT_EULA=Y apt-get install -y mssql-tools
RUN echo 'export PATH="$PATH:/opt/mssql-tools/bin"' >> ~/.bashrc
RUN bash -c 'echo "export PATH=\"$PATH:/opt/mssql-tools/bin\"" >> ~/.bashrc'
RUN apt-get install -y unixodbc-dev

RUN pecl install sqlsrv \
	&& pecl install pdo_sqlsrv \
	&& docker-php-ext-enable sqlsrv pdo_sqlsrv
RUN sed -i '$s/CipherString = DEFAULT@SECLEVEL=2/CipherString = DEFAULT@SECLEVEL=1/' /etc/ssl/openssl.cnf

# install nodejs
RUN apt-get update && \
    apt-get install -y nodejs npm && \
	npm install pm2 -g

# Install supervisord
RUN apt-get install -y supervisor

RUN a2enmod rewrite && a2enmod ssl && a2enmod socache_shmcb
RUN mkdir /var/www/support.dev.sdi
RUN sed -i '/<Directory \/var\/www\/>/,/<\/Directory>/ s/AllowOverride None/AllowOverride All/' /etc/apache2/apache2.conf
COPY ./certs/support.crt /etc/ssl/certs/
COPY ./certs/support.key /etc/ssl/private/
COPY ./apache/support.dev.sdi.conf /etc/apache2/sites-available/
RUN a2ensite support.dev.sdi.conf
RUN service apache2 restart
# RUN apt-get update && apt-get upgrade -y

# Copy the supervisord configuration
COPY supervisord.conf /etc/supervisor/conf.d/supervisord.conf

RUN mv "$PHP_INI_DIR/php.ini-production" "$PHP_INI_DIR/php.ini"

# Start supervisord to manage both Apache and Node.js
CMD ["supervisord", "-c", "/etc/supervisor/conf.d/supervisord.conf"]
