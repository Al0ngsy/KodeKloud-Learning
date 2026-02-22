# 12-Factor Application Methodology

## 1. Codebase

Use **Git** to track your codebase, and have a **single codebase per app/service**.

- Multiple deploys from the same codebase are allowed (e.g., staging and production), but there should be a one-to-one relationship between codebase and app/service.
- Each codebase should be tracked in a revision control system, such as Git, Mercurial, or Subversion.
- Central repositories that support storing codebases: Github, Gitlab, Bitbucket, and Azure Repos.

Example Git commands:
```bash
# Initialize a new Git repository
git init

# Add files to the staging area
git add .

# Commit changes with a message
git commit -m "Initial commit"

# Push to a remote repository
git remote add origin <repository_url>
git push -u origin master
```

Or using Git GUI tools like GitHub Desktop, GitKraken, SourceTree or Git Graph Extension on VS Code can also help to manage the codebase visually.

## 2. Dependencies

**Don't rely on implicit existence of system-wide packages.** Declare all dependencies explicitly in a manifest file:

- `requirements.txt` for Python
- `package.json` for Node.js
- `Gemfile` for Ruby

Use a **dependency manager** to handle the installation and versioning of these dependencies.

**Version control** (by version numbers) and **isolation** (using virtual environments like `venv` for Python or containers like Docker) are crucial to ensure consistent app execution across environments.

Example Dockerfile for a Python application:
```Dockerfile
# Use an official Python runtime as a parent image
FROM python:3.10-alpine

# Set the working directory in the container
WORKDIR /app

# Copy the requirements file into the container at /app
COPY requirements.txt /app

# Install any needed packages specified in requirements.txt
RUN pip install --no-cache-dir -r requirements.txt

# Copy the rest of the application code into the container
COPY . /app

# Run the application
CMD python app.py
```

Docker commands to build and run the container:
```bash
# Build the Docker image
docker build -t my-python-app .

# Run the Docker container
docker run -d -p 5000:5000 my-python-app
```

## 3. Config

**Configuration** refers to the settings and parameters that determine how an app behaves in different environments.

- Store settings in **environment variables** (e.g., `.env` files or k8s ConfigMaps) or configuration files, rather than hardcoding into the app's code.
- This allows for greater flexibility and easier management of different environments (development, staging, production).

## 4. Backing Services

**Backing services** are any services that the app consumes over the network as part of its normal operation.

- Examples: databases, message queues, caching systems, external APIs
- Treat services as **attached resources** that can be swapped out without requiring code changes.
- Connect using **configuration** (e.g., environment variables) rather than hardcoding connection details.

Example of Backing Services:
- **Database**: PostgreSQL, MySQL, MongoDB
- **Message Queue**: RabbitMQ, Apache Kafka
- **Caching System**: Redis, Memcached

5. Build, release, run

Clear separation between the build, release, and run stages of the app's lifecycle is essential.

- **Build stage**: Compiling the code and packaging it into a deployable artifact (e.g., a Docker image, a JAR file, or a ZIP file).
- **Release stage**: Combining the build artifact with the configuration to create a release that can be deployed to production (e.g., using a CI/CD pipeline, Docker Container + .env files). Each release should be uniquely identifiable (e.g., by a version number or a Git commit hash) and should be immutable, meaning that once a release is created, it cannot be modified. This way it can be easily rolled back to a previous release if needed.
- **Run stage**: Executing the app in the defined environment (dev, staging, production with its own environment variables).

## 6. Processes

### Stateless Processes

- Design the app to be **stateless**, so any instance can handle any request without relying on previous interactions.
- This enables better **scalability and resilience**, as any instance can be replaced or scaled without affecting functionality.
- **Stateful data** should be stored in a backing service (database, cache) accessible by any app instance.

## 7. Port Binding

The app should be **self-contained** and should not rely on any external web server to run.

- Bind to a specific port and listen for incoming requests.
- This enables easy deployment and scaling in any environment supporting the required runtime (container, virtual machine, serverless platform).

## 8. Concurrency

Each instance of the app can serve multiple requests. On high influx of requests, scale up by:

- **Vertical scaling**: Increase resources (may require restart and cause downtime)
- **Horizontal scaling**: Run multiple app instances

**Horizontal scaling** requires:
- A **load balancer** to distribute traffic across instances (e.g., Nginx, HAProxy, AWS/Azure/Google Cloud load balancers)
- **Stateless** app design so any instance can handle any request without relying on previous interactions

## 9. Disposability

The app should be designed to **start up and shut down quickly**, allowing for rapid scaling and deployment.

- Processes should be **disposable** — they can be started or stopped at any time without affecting overall functionality.
- This enables easier failure recovery — any instance can be replaced without impacting the system.
- Example: `docker stop ...` sends SIGTERM signal for graceful shutdown. If not shutdown within a timeframe, SIGKILL forcefully terminates. 

## 10. Dev/Prod Parity

**Development, staging, and production environments should be as similar as possible** to reduce bugs and issues from environment differences.

This includes:
- Same backing services (databases, message queues) in the same version/schema
- Same configuration management (environment variables with same keys but different values per environment)
- Same deployment processes (CI/CD pipelines) across all environments
- Feature developers participate in the deployment process to ensure issues can be easily reproduced and fixed 

## 11. Logs

The app should **treat logs as event streams** and write to permanent storage, not local storage.

- The app should **not** be responsible for managing log files or log rotation.
- Send logs to a **centralized logging system** (e.g., ELK Stack, Splunk, Datadog).
- This enables better monitoring and troubleshooting by aggregating and analyzing logs from all app instances.

## 12. Admin Processes

**Admin processes** are one-off tasks run in the same environment as the app, but not part of normal operation.

**Examples:**
- Database migrations
- Data seeding
- Running a console for debugging

Run using the **same codebase, configuration, and backing services** as the app to ensure consistency and reduce environment-related issues. 
