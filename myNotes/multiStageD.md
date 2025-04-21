```dockerfile
# --- Builder Stage ---
FROM python:3.12-slim-buster AS builder

WORKDIR /app

COPY requirements.txt .

# Install dependencies
RUN pip install --no-cache-dir -r requirements.txt

# Copy application code
COPY . .

# --- Final Stage ---
FROM python:3.12-slim-buster

WORKDIR /app

# Copy only the necessary artifacts from the builder stage
COPY --from=builder /app .

# Define the command to run the application
CMD ["python", "app.py"]
```

**Explanation of the Multistage Dockerfile:**

1.  **`FROM python:3.12-slim-buster AS builder`**:
    * This starts the first stage of the build, named `builder`.
    * It uses the same Python base image as before.
    * All the steps in this stage (setting the working directory, copying requirements, installing dependencies, and copying the application code) are performed within this temporary `builder` image.

2.  **`WORKDIR /app` (in builder stage)**: Sets the working directory for the builder stage.

3.  **`COPY requirements.txt .` (in builder stage)**: Copies the requirements file.

4.  **`RUN pip install --no-cache-dir -r requirements.txt` (in builder stage)**: Installs the dependencies. These dependencies are only needed during the build process to create the necessary Python packages.

5.  **`COPY . .` (in builder stage)**: Copies the application code.

6.  **`FROM python:3.12-slim-buster`**:
    * This starts the second and final stage of the build.
    * It again uses a minimal Python base image. This will be the image that is eventually deployed.

7.  **`WORKDIR /app` (in final stage)**: Sets the working directory for the final image.

8.  **`COPY --from=builder /app .`**:
    * This is the key line for the multistage build. It copies the contents of the `/app` directory from the `builder` stage into the `/app` directory of the current (final) stage.
    * **Crucially, only the necessary artifacts (your application code and the installed dependencies in the site-packages directory) are copied over.** Any intermediate files or build tools used in the `builder` stage are not included in the final image.

9.  **`CMD ["python", "app.py"]` (in final stage)**: Defines the command to run the application in the final image.

**How this reduces image size:**

* **Separation of Concerns:** The `builder` stage handles all the build-time dependencies and operations. The final stage only contains the minimal runtime environment and your application.
* **Reduced Layers:** The final image doesn't include the extra layers created by installing build tools, intermediate files, or the entire build environment.
* **Smaller Final Image:** By copying only the essential artifacts, the final Docker image becomes significantly smaller, leading to faster deployments, less storage usage, and improved security.

**To use this multistage Dockerfile:**

The build and run commands remain the same:

```bash
docker build -t my-python-app .
docker run my-python-app
```

Docker will automatically execute both stages defined in the Dockerfile and create the final, smaller image.
