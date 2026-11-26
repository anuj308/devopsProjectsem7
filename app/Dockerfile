FROM nginx:1.28-alpine
COPY index.html styles.css /usr/share/nginx/html/
RUN chmod 0644 /usr/share/nginx/html/index.html /usr/share/nginx/html/styles.css
HEALTHCHECK --interval=10s --timeout=3s --retries=3 CMD wget -q -O /dev/null http://127.0.0.1/ || exit 1
