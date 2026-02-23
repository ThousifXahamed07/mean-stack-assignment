# Stage 1: Build Angular application
FROM node:18-alpine AS builder

# Set working directory
WORKDIR /app

# Copy package files
COPY package.json package-lock.json* ./

# Install dependencies
RUN npm install

# Copy application source code
COPY . .

# Build the Angular app - replaced 'npm run build --prod' with the direct CLI command
RUN npx -p @angular/cli ng build --configuration production

# Stage 2: Serve with Nginx
FROM nginx:alpine

# Remove default nginx configuration
RUN rm -rf /usr/share/nginx/html/*

# Copy built application from builder stage
# Angular CLI outputs to dist/<project-name> by default
COPY --from=builder /app/dist/angular-15-crud /usr/share/nginx/html

# Copy custom nginx configuration
COPY nginx.conf /etc/nginx/conf.d/default.conf

# Expose port 80
EXPOSE 80

# Add healthcheck
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD wget --quiet --tries=1 --spider http://localhost/ || exit 1

# Start nginx
CMD ["nginx", "-g", "daemon off;"]
