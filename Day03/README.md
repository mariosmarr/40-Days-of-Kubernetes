# A sample todo app in react
# Day 3: Multi-Stage Docker Builds & Best Practices

### What I accomplished today
Today was all about optimizing Docker images for production. Instead of packing the whole development environment (Node.js, source code, and heavy `node_modules`) into the final container, I learned how to use a technique called **Multi-Stage Build**. I containerized a React/Node app, resulting in a super lightweight and secure image.

### The Magic of Multi-Stage Builds
I wrote a single `Dockerfile` divided into two completely separate stages:
- **Stage 1 (Installer/Builder):** I used a Node.js image to copy the `package.json`, install all dependencies, and run the build command to generate the static website files.
- **Stage 2 (Deployer):** I started a completely fresh, empty container using `nginx` (a fast web server). I copied *only* the finished static files from Stage 1 into the Nginx web root directory. 

**The result:** Docker automatically discards the entire Stage 1 (Node.js, source code, tools). The final image is drastically smaller and much more secure because it only contains a web server and the ready-to-serve website files.

### Docker Best Practices I Learned
Reviewing the official Docker documentation, I noted down some golden rules for writing a good Dockerfile:
- **Use lightweight base images:** Like the `alpine` versions (e.g., `node:18-alpine`), which keep the image size minimal.
- **Run as a non-root user:** By default, containers run as `root`. For security, it's a best practice to configure the container to run as an unprivileged user to prevent serious breaches.
- **Use a `.dockerignore` file:** Just like `.gitignore`, this prevents heavy or sensitive local files (like local `node_modules` or `.env` files) from being copied into the container during the build process.

### Exploring the `docker init` command
I tested the `docker init` command in an empty folder. It is a fantastic shortcut! It asks you a few simple questions in the terminal (like what programming language your project uses) and automatically generates a basic `Dockerfile`, a `.dockerignore`, and a `compose.yaml` file to help you get started without writing everything from scratch.

### Essential Commands & Troubleshooting
- **`docker exec -it <container_id> sh`**: This command lets me "SSH" into a running container and open an interactive terminal. I used it to explore the internal folder structure like `/var/log/nginx`.
- **`docker logs <container_name>`**: While exploring inside the container, I realized that reading log files (like `access.log` and `error.log`) with the `cat` command can freeze the terminal. This is because Nginx maps these logs directly to the system's output streams (`stdout`/`stderr`). I learned that the correct and safe way to check logs is to stay outside the container and simply run the `docker logs` command.