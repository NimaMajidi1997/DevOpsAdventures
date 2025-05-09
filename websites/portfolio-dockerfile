FROM nginx:alpine
RUN apk update && apk add git
RUN rm -rf /usr/share/nginx/html/* \
    && git clone https://github.com/NimaMajidi1997/devportfolio.git /usr/share/nginx/html