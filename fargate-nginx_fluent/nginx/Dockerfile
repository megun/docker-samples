FROM nginx:alpine

COPY ./etc/nginx/nginx.conf /etc/nginx/nginx.conf
COPY ./etc/nginx/conf.d/default.conf /etc/nginx/conf.d/default.conf
COPY ./usr/share/nginx/html /usr/share/nginx/html

RUN mkdir /data1 \
  && ln -sfn /data1/access.log /var/log/nginx/access.log \
  && ln -sfn /data1/error.log /var/log/nginx/error.log \
  && touch /data1/access.log \
  && touch /data1/error.log
