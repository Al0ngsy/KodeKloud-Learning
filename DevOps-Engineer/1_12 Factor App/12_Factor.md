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
4. Backing Services
5. Build, release, run
6. Processes
7. Port Binding
8. Concurrency
9. Disposability
10. Dev/prod parity
11. Logs
12. Admin Processes
