# Serve static files with Nginx on Alpine
FROM alpine:3.18

# Install nginx
RUN apk add --no-cache nginx

# Remove default config and replace with one that serves static files
RUN rm /etc/nginx/http.d/default.conf

# Add custom nginx config
RUN echo 'server { \
    listen 80 default_server; \
    root /usr/share/nginx/html; \
    index index.html; \
    location / { \
        try_files $uri $uri/ =404; \
    } \
}' > /etc/nginx/http.d/default.conf

# Copy static files to nginx html directory
COPY html/ /usr/share/nginx/html

# Expose port 80
EXPOSE 80

# Start nginx in the foreground
CMD ["nginx", "-g", "daemon off;"]