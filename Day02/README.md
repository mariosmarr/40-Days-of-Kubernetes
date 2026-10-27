# Day 2: Dockerizing a Node.js Application

### What I accomplished today
Today was all about taking an existing application (a simple Node.js Weather App) and making it run independently inside a container. I didn't write the app itself, but my job was to package it so it can run smoothly on any machine.

### The Dockerfile & Fixing Dependencies
I created the `Dockerfile` from scratch. The interesting part was that the original application code was missing a dependency (the `ejs` module). Instead of messing with the developer's source code, I learned that we can fix these things at the infrastructure level. I just added `RUN npm install ejs` inside my Dockerfile, and the container patched the issue automatically during the build!

### Essential Commands I Used
* **`docker build -t <image-name> .`** : This reads the Dockerfile in my current folder (that's what the `.` means) and builds the actual image.
* **`docker tag <image> <username/image>`** : I used this to label my image properly so Docker knows it belongs to my Docker Hub account.
* **`docker push <username/image>`** : This uploads my finished image to the Docker Hub registry so it's safely stored in the cloud.
* **`docker run -dp 3000:3000 <image>`** : This takes the image and spins up a live container. The `-d` runs it in the background, and `-p` connects my computer's port 3000 to the container's port 3000 so I can see the app in my browser.

### Troubleshooting & Cool Tools
* **`docker init`**: I discovered this handy shortcut command. It asks a few simple questions in the terminal and automatically generates a basic Dockerfile to start with.
* **WSL DNS Fix**: I ran into a bug where my WSL environment couldn't connect to the internet to download Docker images. I fixed it by going into the Docker Engine settings, hardcoding the DNS server to `8.8.8.8`, and running `wsl --shutdown` to restart the system.