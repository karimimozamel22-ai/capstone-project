FROM rockylinux:9

# Install Apache + PHP
RUN dnf -y update && \
    dnf -y install httpd php php-mysqlnd && \
    dnf clean all

# Override php.conf to use mod_php instead of php-fpm
RUN echo '<FilesMatch \.php$>' > /etc/httpd/conf.d/php.conf && \
    echo '    SetHandler application/x-httpd-php' >> /etc/httpd/conf.d/php.conf && \
    echo '</FilesMatch>' >> /etc/httpd/conf.d/php.conf

# Remove fpm and welcome page configs
RUN rm -f /etc/httpd/conf.d/php-fpm.conf && \
    rm -f /etc/httpd/conf.d/welcome.conf

# Download and extract app
RUN curl -o /tmp/lab-app.tgz https://aws-tc-largeobjects.s3-us-west-2.amazonaws.com/CUR-TF-200-ACACAD/studentdownload/lab-app.tgz && \
    tar -xvzf /tmp/lab-app.tgz -C /var/www/html/ && \
    rm -f /tmp/lab-app.tgz

# Set permissions
RUN chown -R apache:apache /var/www/html && \
    chown apache:root /var/www/html/rds.conf.php

# Fix Apache ServerName warning
RUN echo "ServerName localhost" >> /etc/httpd/conf/httpd.conf

EXPOSE 80

CMD ["/usr/sbin/httpd", "-D", "FOREGROUND"]
