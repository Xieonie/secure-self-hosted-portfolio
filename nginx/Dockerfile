# Use the official Nginx image from Docker Hub
FROM nginx:alpine

# Remove default Nginx configuration file
RUN rm /etc/nginx/conf.d/default.conf

# Copy custom Nginx configuration file
COPY nginx.conf /etc/nginx/nginx.conf

# Optional: Copy any static assets that should be part of the image itself
# (though mounting via docker-compose.yml is usually preferred for website content)
# COPY ./my-custom-error-pages /usr/share/nginx/html/error-pages

# Expose port 80 (Nginx default)
EXPOSE 80

# Command to run Nginx in the foreground
CMD ["nginx", "-g", "daemon off;"]