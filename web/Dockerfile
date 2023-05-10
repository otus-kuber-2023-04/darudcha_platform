FROM nginx:alpine
RUN  mkdir /app && touch /app/work.html && chown -R 1001:1001 /app \
     && echo '<html>\n\t<body>\n\t\t<h1>Hello World NGINX!</h1>\n\t</body>\n</html>' > /app/work.html \
     && cp /app/work.html /usr/share/nginx/html
#COPY /app/work.html /usr/share/nginx/html
EXPOSE 80/tcp
WORKDIR /usr/share/nginx/html
VOLUME /app
CMD ["/usr/sbin/nginx", "-g", "daemon off;"]
RUN chown -R 1001:1001 /usr/share/nginx
RUN chown -R 1001:1001 /var/log/nginx
RUN touch /run/nginx.pid
RUN chown -R 1001:1001 /run/nginx.pid
RUN chown -R 1001:1001 /var/cache/nginx
RUN chown -R 1001:1001 /etc/nginx
USER 1001
