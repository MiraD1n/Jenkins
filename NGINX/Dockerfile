# src: build/backend/Dockerfile

# Используем за основу контейнера Ubuntu 14.04 LTS
FROM ubuntu:latest
# Переключаем Ubuntu в неинтерактивный режим — чтобы избежать лишних запросов
ENV DEBIAN_FRONTEND noninteractive
RUN apt-get update \
&& apt-get install -y cpp wget libpcre3-dev libc6-dev zlib1g-dev gcc make \
&& mkdir -p ~/temp/nginx-install \
&& cd ~/temp/nginx-install \
&& wget http://nginx.org/download/nginx-1.7.7.tar.gz \
&& tar -xvf nginx-1.7.7.tar.gz \
&& cd ./nginx-1.7.7 \
&& ./configure --with-cc-opt="-Wno-error" \
&& make \
&& make install \ 
&& wget https://raw.githubusercontent.com/JasonGiedymin/nginx-init-ubuntu/master/nginx -O /etc/init.d/nginx \
&& chmod +x /etc/init.d/nginx
VOLUME ["/var/www"]
EXPOSE 80
