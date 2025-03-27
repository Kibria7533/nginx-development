FROM nginx:alpine

# Remove the default nginx index page
RUN rm -rf /usr/share/nginx/html/*

# Copy custom HTML to NGINX web root
COPY index.html /usr/share/nginx/html/index.html
