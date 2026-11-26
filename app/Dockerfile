FROM nginx:1.28-alpine
COPY --chmod=0644 index.html styles.css /usr/share/nginx/html/
HEALTHCHECK --interval=10s --timeout=3s --retries=3 CMD wget -q -O /dev/null http://127.0.0.1/ || exit 1
