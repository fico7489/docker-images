FROM nginx:1.19.2-alpine

ARG NGINX_PATH
ENV NGINX_PATH=${NGINX_PATH}

COPY ${NGINX_PATH}/nginx.conf /etc/nginx/conf.d/default.conf
