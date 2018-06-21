# Dockerfile - Debian Jessie
# https://github.com/openresty/docker-openresty

FROM debian:jessie

# Docker Build Arguments
ARG RESTY_VERSION="1.13.6.1"
ARG RESTY_LUAROCKS_VERSION="2.4.3"
ARG RESTY_OPENSSL_VERSION="1.0.2k"
ARG RESTY_PCRE_VERSION="8.41"
ARG RESTY_J="1"
ARG RESTY_CONFIG_OPTIONS="\
    --with-file-aio \
    --with-http_addition_module \
    --with-http_auth_request_module \
    --with-http_dav_module \
    --with-http_flv_module \
    --with-http_gunzip_module \
    --with-http_gzip_static_module \
    --with-http_image_filter_module=dynamic \
    --with-http_mp4_module \
    --with-http_random_index_module \
    --with-http_realip_module \
    --with-http_secure_link_module \
    --with-http_slice_module \
    --with-http_ssl_module \
    --with-http_stub_status_module \
    --with-http_sub_module \
    --with-http_v2_module \
    --with-http_xslt_module=dynamic \
    --with-ipv6 \
    --with-mail \
    --with-mail_ssl_module \
    --with-md5-asm \
    --with-pcre-jit \
    --with-sha1-asm \
    --with-stream \
    --with-stream_ssl_module \
    --with-threads \
    "
ARG RESTY_CONFIG_OPTIONS_MORE=""

ENV RESTY_CONFIG_OPTIONS_MORE="$RESTY_CONFIG_OPTIONS_MORE --add-module=/tmp/ngx_http_geoip2_module"

# These are not intended to be user-specified
ARG _RESTY_CONFIG_DEPS="--with-openssl=/tmp/openssl-${RESTY_OPENSSL_VERSION} --with-pcre=/tmp/pcre-${RESTY_PCRE_VERSION}"

ARG GEOIP_LICENSE

ENV GEOIP_DIR /usr/local/share/GeoIP

# 1) Install apt dependencies
# 2) Download and untar OpenSSL, PCRE, and OpenResty
# 3) Build OpenResty
# 4) Cleanup

RUN DEBIAN_FRONTEND=noninteractive apt-get update \
  && DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
  git \
  make \
  build-essential \
  ca-certificates \
  curl \
  gettext-base \
  libgd-dev \
  libncurses5-dev \
  libperl-dev \
  libreadline-dev \
  libxslt1-dev \
  perl \
  unzip \
  zlib1g-dev \
  autoconf \
  automake \
  procps \
  dh-autoreconf

# Download GeoIP2 Nginx Module
RUN git clone https://github.com/leev/ngx_http_geoip2_module.git /tmp/ngx_http_geoip2_module

# Install C library for reading MaxMind DB files
# Resource: https://github.com/maxmind/libmaxminddb
RUN git clone --recursive https://github.com/maxmind/libmaxminddb.git && \
  cd libmaxminddb && \
  ./bootstrap && \
  ./configure && \
  make && \
  make check && \
  make install && \
  echo /usr/local/lib  >> /etc/ld.so.conf.d/local.conf && \
  ldconfig

RUN rm -rf libmaxminddb

# Download/install geoip databases
###
RUN mkdir -p $GEOIP_DIR
RUN curl -K -L http://geolite.maxmind.com/download/geoip/database/GeoLite2-City.tar.gz -o GeoLite2-City.tar.gz
RUN GEOLITE_CITY=`tar -xzvf GeoLite2-City.tar.gz | head -1 | cut -f1 -d"/"` && \
  tar -xzvf GeoLite2-City.tar.gz && \
  mv $GEOLITE_CITY/GeoLite2-City.mmdb $GEOIP_DIR/GeoLite2-City.mmdb

RUN curl -k -L "https://download.maxmind.com/app/geoip_download?edition_id=GeoIP2-Connection-Type&suffix=tar.gz&license_key=${GEOIP_LICENSE}" -o GeoIP2-Connection-Type.tar.gz
RUN GEOIP_CONN=`tar -xzvf GeoIP2-Connection-Type.tar.gz | head -1 | cut -f1 -d"/"` && \
tar -xzvf GeoIP2-Connection-Type.tar.gz && \
rm GeoIP2-Connection-Type.tar.gz && mv $GEOIP_CONN/GeoIP2-Connection-Type.mmdb $GEOIP_DIR/GeoIP2-Connection-Type.mmdb

RUN cd /tmp \
    && curl -fSL https://www.openssl.org/source/openssl-${RESTY_OPENSSL_VERSION}.tar.gz -o openssl-${RESTY_OPENSSL_VERSION}.tar.gz \
    && tar xzf openssl-${RESTY_OPENSSL_VERSION}.tar.gz \
    && curl -fSL https://ftp.pcre.org/pub/pcre/pcre-${RESTY_PCRE_VERSION}.tar.gz -o pcre-${RESTY_PCRE_VERSION}.tar.gz \
    && tar xzf pcre-${RESTY_PCRE_VERSION}.tar.gz \
    && curl -fSL https://openresty.org/download/openresty-${RESTY_VERSION}.tar.gz -o openresty-${RESTY_VERSION}.tar.gz \
    && tar xzf openresty-${RESTY_VERSION}.tar.gz \
    && cd /tmp/openresty-${RESTY_VERSION} \
    && ./configure -j${RESTY_J} ${_RESTY_CONFIG_DEPS} ${RESTY_CONFIG_OPTIONS} ${RESTY_CONFIG_OPTIONS_MORE} \
    && make -j${RESTY_J} \
    && make -j${RESTY_J} install \
    && cd /tmp \
    && rm -rf \
        openssl-${RESTY_OPENSSL_VERSION} \
        openssl-${RESTY_OPENSSL_VERSION}.tar.gz \
        openresty-${RESTY_VERSION}.tar.gz openresty-${RESTY_VERSION} \
        pcre-${RESTY_PCRE_VERSION}.tar.gz pcre-${RESTY_PCRE_VERSION} \
    && curl -fSL https://github.com/luarocks/luarocks/archive/${RESTY_LUAROCKS_VERSION}.tar.gz -o luarocks-${RESTY_LUAROCKS_VERSION}.tar.gz \
    && tar xzf luarocks-${RESTY_LUAROCKS_VERSION}.tar.gz \
    && cd luarocks-${RESTY_LUAROCKS_VERSION} \
    && ./configure \
        --prefix=/usr/local/openresty/luajit \
        --with-lua=/usr/local/openresty/luajit \
        --lua-suffix=jit-2.1.0-beta3 \
        --with-lua-include=/usr/local/openresty/luajit/include/luajit-2.1 \
    && make build \
    && make install \
    && cd /tmp \
    && rm -rf luarocks-${RESTY_LUAROCKS_VERSION} luarocks-${RESTY_LUAROCKS_VERSION}.tar.gz \
    && DEBIAN_FRONTEND=noninteractive apt-get autoremove -y \
    && ln -sf /dev/stdout /usr/local/openresty/nginx/logs/access.log \
    && ln -sf /dev/stderr /usr/local/openresty/nginx/logs/error.log \
    && rm -rf /var/lib/apt/lists/* \
    && rm -rf /tmp/ngx_http_geoip2_module

# Add additional binaries into PATH for convenience
ENV PATH=$PATH:/usr/local/openresty/luajit/bin:/usr/local/openresty/nginx/sbin:/usr/local/openresty/bin

# Copy nginx configuration files
COPY nginx.conf /usr/local/openresty/nginx/conf/nginx.conf
COPY nginx.vh.default.conf /etc/nginx/conf.d/default.conf

# TODO: remove any other apt packages?
CMD ["/usr/local/openresty/bin/openresty", "-g", "daemon off;"]
