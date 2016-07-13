FROM centos:centos7
MAINTAINER Luke Flynn <luke@lukehflynn.com>

WORKDIR /tmp

# Update Image
RUN yum update -y

# Install prerequisites for Nginx compile
RUN yum install -y \
        wget \
        tar \
        openssl-devel \
        gcc \
        gcc-c++ \
        make \
        zlib-devel \
        pcre-devel \
        gd-devel \
        git

# Download Nginx and Nginx modules source
RUN mkdir -p /usr/local/src/nginx && \
    wget http://nginx.org/download/nginx-1.10.1.tar.gz -O nginx.tar.gz && \
    tar -xzvf nginx.tar.gz -C /usr/local/src/nginx &&\
    git clone https://github.com/pagespeed/ngx_pagespeed /usr/local/src/nginx/ngx_pagespeed && \
    wget https://dl.google.com/dl/page-speed/psol/1.11.33.2.tar.gz -O psol.tar.gz && \
    tar -xzvf psol.tar.gz -C /usr/local/src/nginx/ngx_pagespeed

# Build Nginx
WORKDIR /usr/local/src/nginx/nginx-1.10.1
RUN ./configure \
        --user=nginx \
        --group=nginx \
        --prefix=/usr/share/nginx \
        --sbin-path=/usr/sbin/nginx \
        --conf-path=/etc/nginx/nginx.conf \
        --pid-path=/run/nginx.pid \
        --lock-path=/run/lock/subsys/nginx \
        --error-log-path=/var/log/nginx/error.log \
        --http-log-path=/var/log/nginx/access.log \
        --with-http_gzip_static_module \
        --with-http_ssl_module \
        --with-pcre \
        --with-http_image_filter_module \
        --with-file-aio \
        --with-ipv6 \
        --with-http_v2_module \
        --with-http_gunzip_module \
        --add-module=/usr/local/src/nginx/ngx_pagespeed && \
    make && \
    make install

WORKDIR /usr/local/src/nginx

# Add nginx user
RUN adduser -c "Nginx user" nginx && \
    setcap cap_net_bind_service=ep /usr/sbin/nginx

RUN touch /run/nginx.pid

RUN rm -rf /etc/nginx/nginx.conf

RUN mkdir /var/pagespeed_cache && mkdir /var/www && mkdir /etc/nginx/conf.d

RUN chown nginx:nginx /etc/nginx /var/log/nginx /usr/share/nginx /run/nginx.pid /var/pagespeed_cache /var/www /etc/nginx/conf.d
# Cleanup after Nginx build
RUN yum remove -y \
        wget \
        tar \
        gcc \
        gcc-c++ \
        make \
        git && \
    yum autoremove -y && \
    rm -rf /tmp/*

# PORTS
EXPOSE 80
EXPOSE 443

USER nginx
CMD ["/usr/sbin/nginx", "-g", "daemon off;"]
