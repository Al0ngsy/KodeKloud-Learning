1. Codebase

Use Git to track your codebase, and have a single codebase per app/service.
Multiple deploys from the same codebase are allowed (e.g., staging and production), but there should be a one-to-one relationship between codebase and app/service. 
Each codebase should be tracked in a revision control system, such as Git, Mercurial, or Subversion.
Central Repositories that support storing codebases are Github, Gitlab, Bitbucket, and Azure Repos.

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

2. Dependencies

Don't rely on implicit existence of system-wide packages. Declare all dependencies explicitly in a manifest file (e.g., `requirements.txt` for Python, `package.json` for Node.js, `Gemfile` for Ruby). Use a dependency manager to handle the installation and versioning of these dependencies.

Version control (by version numbers) and isolation (using virtual environments, such as `venv` for Python or containers, such as Docker) are crucial to ensure that the app runs consistently across different environments.

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

3. Config

Configuration refers to the settings and parameters that determine how an app behaves in different environments. These settings should be stored in environment variables (e.g., `.env` files or k8s ConfigMaps) or configuration files, rather than being hardcoded into the app's code. This allows for greater flexibility and easier management of different environments (e.g., development, staging, production).

4. Backing Services

Backing services are any services that the app consumes over the network as part of its normal operation. Examples include databases, message queues, caching systems, and external APIs. These services should be treated as attached resources, meaning they can be swapped out without requiring changes to the app's code. The app should be designed to connect to these services using configuration, such as environment variables, rather than hardcoding connection details.

Example of Backing Services:
- **Database**: PostgreSQL, MySQL, MongoDB
- **Message Queue**: RabbitMQ, Apache Kafka
- **Caching System**: Redis, Memcached

5. Build, release, run

Clear separation between the build, release, and run stages of the app's lifecycle is essential.

- **Build stage**: Compiling the code and packaging it into a deployable artifact (e.g., a Docker image, a JAR file, or a ZIP file).
- **Release stage**: Combining the build artifact with the configuration to create a release that can be deployed to production (e.g., using a CI/CD pipeline, Docker Container + .env files). Each release should be uniquely identifiable (e.g., by a version number or a Git commit hash) and should be immutable, meaning that once a release is created, it cannot be modified. This way it can be easily rolled back to a previous release if needed.
- **Run stage**: Executing the app in the defined environment (dev, staging, production with its own environment variables).

6. Processes

**Stateless processes**: The app should be designed to be stateless, meaning that any instance of the app can handle any request without relying on previous interactions. This allows for better scalability and resilience, as any instance can be replaced or scaled up/down without affecting the overall functionality of the app.

Stateful data should be stored in a backing service, such as a database or a cache, which can be accessed by any instance of the app.

7. Port Binding

The app should be self-contained and should not rely on any external web server to run. Instead, it should bind to a specific port and listen for incoming requests. This allows the app to be easily deployed and scaled, as it can run in any environment that supports the required runtime (e.g., a container, a virtual machine, or a serverless platform).

8. Concurrency

Each instance of the app can serve multiple requests. On high influx of requests, we can scale up by increasing resources (vertical scaling), this might require restarting the app and causing downtime, or by running multiple instances of the app (horizontal scaling).

For horizontal scaling, it is required to have a load balancer to distribute incoming traffic across multiple instances of the app. This can be achieved using tools like Nginx, HAProxy, or cloud-based load balancers provided by AWS, Azure, or Google Cloud. Horizontal scaling requires the app to be **stateless**, meaning that any instance of the app can handle any request without relying on previous interactions.

9. Disposability

The app should be designed to start up and shut down quickly, allowing for rapid scaling and deployment. This means that processes should be disposable, meaning they can be started or stopped at any time without affecting the overall functionality of the app. This also allows for easier recovery from failures, as any instance of the app can be replaced without impacting the system.

For example, `docker stop ...` will send a SIGTERM signal to the app, allowing it to gracefully shut down and release any resources it holds before exiting. If the app does not shut down within a certain time frame, it will be forcefully killed with a SIGKILL signal. 

10. Dev/prod parity

Development, staging, and production environments should be as similar as possible to reduce the chances of bugs and issues that arise from differences between these environments. 

This includes using:
- The same type of backing services (e.g., databases, message queues) in the same version/schema
- The same configuration management (e.g., environment variables with same keys but different values for each environment)
- The same deployment processes (e.g., CI/CD pipelines) across all environments. 

Additionally, the developer of the feature should also take part in the deployment process. This helps to ensure that any issues that arise in production can be easily reproduced and fixed in development or staging and reduce the time and effort required to deploy changes to production. 

11. Logs

The app should treat logs as event streams, meaning that logs should be written to a permanent storage, and should not be stored locally. The app itself should not be responsible for managing log files or log rotation. Instead, logs should be sent to a centralized logging system (e.g., ELK Stack, Splunk, Datadog) that can aggregate and analyze logs from multiple instances of the app. This allows for better monitoring and troubleshooting of the app.

12. Admin Processes

Admin processes are one-off tasks that are run in the same environment as the app, but are not part of the normal operation of the app.

Examples include:
- Database migrations
- Data seeding
- Running a console for debugging purposes 

These processes should be run using the same codebase, configuration, and backing services as the app itself to ensure consistency and reduce the chances of issues arising from differences between environments. 
