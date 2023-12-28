# Use the latest Alpine base image
FROM alpine:latest

# Update the package list and install Node.js and NPM
RUN apk update && apk add nodejs npm

RUN node --version && npm --version


# Create a directory to copy your Node.js application into
WORKDIR /app

# Copy your Node.js application files into the container's /app directory
COPY . /app

# Replace this with your actual command to start your Node.js application
CMD ["node", "app.js"]
