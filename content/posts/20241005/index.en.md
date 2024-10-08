---
title: "Dev Containers"
subtitle: "A Clean, Shareable and Consistent Development Environment"
date: 2024-10-05T16:55:25+03:00
draft: false

tags: ["mlops", "docker", "dev-containers", "vscode"]
categories: ["Development Tools", "MLOps", "Engineering", "Productivity"]
---

If you're like me and often juggle projects across different tech stacks—think Scala, Rust, Go, Node, Next.js—you know the pain of managing a multitude of dependencies on your local computer. While Python’s virtual environments offer some relief, other languages can turn your system into a dependency jungle.

I used to rely on Docker to handle dependencies by mounting local directories as volumes. While that worked, my experience was less than ideal—especially when it came to IDE integration. Everything still felt tightly coupled to my local machine.

Then I discovered Dev Containers, and everything changed.

## What are Dev Containers?

Dev Containers are the baby of VSCode and Docker. By configuring a simple JSON file called .devcontainer.json, you can define:

- **Base Image:** The Docker image that forms the foundation of your environment.
- **Features:** Additional tools, extensions, or languages your project needs.
- **Ports:** Network settings and exposed ports for communication between services.

Once set up, VSCode detects this file and offers to spin up a Docker container pre-configured with everything you've specified. The magic? Your IDE connects directly to this container, allowing you to code inside a controlled and reproducible environment.

{{< admonition type=example title="Example 1: Dev Container for a Node.js" open=true >}}
Let’s say you’re working on a Node.js project. You can define a simple .devcontainer.json like this:

```json
{
  "name": "Node.js Dev Container",
  "image": "mcr.microsoft.com/vscode/devcontainers/javascript-node:0-14",
  "features": {
    "ghcr.io/devcontainers/features/docker-in-docker:1": {
      "version": "latest"
    }
  },
  "postCreateCommand": "npm install",
  "ports": [3000],
  "settings": {
    "terminal.integrated.shell.linux": "/bin/bash"
  }
}
```

In this example:

- **Base Image:** We're using the official VSCode Node.js image.
- **Features:** Docker-in-Docker allows running Docker commands inside the container (useful for containerized applications).
postCreateCommand: Runs npm install after the container starts to ensure all dependencies are installed.
- **Ports:** Exposes port 3000, useful for local development of web servers.

When you open this project in VSCode, it will automatically prompt you to open it inside the container. This ensures everyone on the team has the same setup!'
{{< /admonition >}}


## Why **You** Should Consider Dev Containers:

1. **Keep Your Local Machine Clean**: Say goodbye to polluting your machine with different versions of tools, libraries, and dependencies. You can close a project and delete the container without leaving a trace on your system. All the configuration and dependencies are stored in the container.
2. **Consistent Development Environments**:  Ever heard "it works on my machine"? Share the .devcontainer.json file with your team, and now everyone will have the exact same environment. This consistency eliminates most setup issues, especially in complex projects.
3. **Easy to Set Up and Share**: Setting up a Dev Container takes only minutes, and once created, it can be shared easily. With just a couple of files in your repo, any developer can clone your project and be ready to go in seconds.

{{< admonition type=example title="Example 2: Setting Up a Python Environment with Dev Containers" open= true >}}
```json
{
  "name": "Python 3 Dev Container",
  "image": "mcr.microsoft.com/vscode/devcontainers/python:3",
  "features": {
    "ghcr.io/devcontainers/features/poetry:1": {}
  },
  "postCreateCommand": "pip install -r requirements.txt",
  "settings": {
    "python.pythonPath": "/usr/local/bin/python"
  },
  "extensions": [
    "ms-python.python"
  ]
}
```

In this case:

- **Base Image:** We're using a Python 3 image.
- **Features:** We include Poetry for Python dependency management.
- **Extensions**: Adds the Python extension to VSCode for a better coding experience.
{{< /admonition >}}


## Bonus Use Cases in My Day-to-Day Work:
Here’s where Dev Containers (and docker) really shine in my work:


1. **On-Demand Tools with Docker Compose**: Sometimes I need specific versions of tools, like AWS CLI or Google Cloud SDK, which aren’t always available via my package manager. Instead of manually installing these, I create a docker-compose.yml file and run them as containers:

```yaml
version: "3.8"
services:
  aws:
    image: amazon/aws-cli
    volumes:
      - .:/workspace
    working_dir: /workspace

```
Now, I can use the AWS CLI by running:

```bash
docker compose run --rm aws <command>
```

The tool runs directly from a container, with no need for local installation. This is perfect for isolating different versions of tools across projects.


2. **Seamless Integration Testing:** When working on microservices or distributed systems, I often spin up multiple services for testing. Using docker-compose.yml, I can create a full-stack testing environment with just a single command:

```yaml
version: "3.8"
services:
  app:
    build: .
    ports:
      - "5000:5000"
  db:
    image: postgres
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
    ports:
      - "5432:5432"
  redis:
    image: redis
    ports:
      - "6379:6379"
```
Running docker compose up brings up the application, database, and cache services, all linked and ready to interact with each other. This setup lets me test integrations in real-time without needing to install anything on my local machine.

## Conclusion

Dev Containers are a game changer for managing dependencies and ensuring consistency across environments. Whether you're working with Node, Python, Rust, Go, or any other stack, Dev Containers simplify the process by encapsulating everything in a clean, shareable environment. No more "works on my machine" headaches—just a smooth, repeatable experience for everyone involved.

---
If you like this post,  please let me know by following me on X or sharing it with your network. I'd love to hear your thoughts! - {{< person url="https://x.com/theshlomigreen" name="Shlomi Green" nick="@theshlomigreen" text="Author" picture="https://pbs.twimg.com/profile_images/1707313015517859840/8lWHx6VR_400x400.jpg" >}}
