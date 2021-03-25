ARG NGINX_VERSION=1.19.8

ARG NGX_BROTLI_COMMIT=9aec15e2aa6feea2113119ba06460af70ab3ea62

FROM ghcr.io/2ndlab/nginx-builder:main AS Builder

RUN \
        apk add --no-cache --virtual .brotli-deps \
            autoconf \
            libtool \
            automake \
            g++ \
            cmake \
        && \
        apk add --no-cache --virtual .utils-deps \
            git 

RUN \
        mkdir -p /usr/src/ngx_brotli && cd /usr/src/ngx_brotli \
        && git init && git remote add origin https://github.com/google/ngx_brotli.git \
        && git fetch --depth 1 origin ${NGX_BROTLI_COMMIT} \
        && git checkout --recurse-submodules -q FETCH_HEAD \
        && git submodule update --init --depth 1 \
        && cd /usr/src/nginx \
        && CONFARGS=$(nginx -V 2>&1 | sed -n -e 's/^.*arguments: //p') \
        CONFARGS=${CONFARGS/-Os -fomit-frame-pointer -g/-Os} \
        && echo $CONFARGS \
        && ./configure --with-compat $CONFARGS --add-dynamic-module=/usr/src/ngx_brotli \
        && make modules

FROM scratch

COPY --from=Builder /usr/src/nginx/objs/ngx_http_brotli_filter_module.so /
COPY --from=Builder /usr/src/nginx/objs/ngx_http_brotli_static_module.so /