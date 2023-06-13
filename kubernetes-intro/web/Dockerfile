FROM nginx:alpine
RUN  mkdir /app && touch /app/index.html && chown -R 1001:1001 /app \
     && echo 'Hello World NGINX!' > /app/index.html
COPY ./config/nginx.conf /etc/nginx/nginx.conf
EXPOSE 8000/tcp
WORKDIR /app
CMD ["/usr/sbin/nginx", "-g", "daemon off;"]
RUN chown -R 1001:1001 /usr/share/nginx
RUN chown -R 1001:1001 /var/log/nginx
RUN touch /run/nginx.pid
RUN chown -R 1001:1001 /run/nginx.pid
RUN chown -R 1001:1001 /var/cache/nginx
RUN chown -R 1001:1001 /etc/nginx
USER 1001
