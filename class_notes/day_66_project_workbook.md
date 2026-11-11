# Day 66 Project Workbook — Containerized ETL Application

## Goal
Package a Python ETL script into a Docker image, run it in a container and preserve output outside the container.

## Step 1 — Prepare the project
Create Dockerfile, requirements.txt, app.py, input/orders.csv and output/ using the class notes.

## Step 2 — Test Python locally
Install the pinned dependency in a suitable virtual environment and run app.py. Confirm city totals: Bengaluru 300, Chennai 400, Hyderabad 400.

## Step 3 — Build the image
Run docker build with a meaningful image name and tag. Record actual output and image ID.

## Step 4 — Run with bind mounts
Mount host input and output folders. Run with --rm. Confirm output/city_sales.csv appears on the host after the container exits.

## Step 5 — Test configuration
Override INPUT_FILE and OUTPUT_FILE with runtime environment variables. Confirm the application reads the supplied paths.

## Step 6 — Test failure behavior
Try a missing input path in a controlled test. Record the error. Restore the correct path and rerun. Do not hide the failure with an empty output.

## Step 7 — Add Compose
Create compose.yaml with one ETL service, build configuration, input/output mounts and non-secret environment settings. Run docker compose up --build and docker compose down.

## Step 8 — Security review
Confirm no passwords are baked into the image, dependencies are pinned, only required folders are mounted and the base image is trusted.

## Submission
Submit project files, Dockerfile, Compose file, build/run commands, actual logs, output CSV, failure evidence, README, Docker/Python versions and validation status.

## Completion criteria
Another student can build the image, run the ETL container, obtain expected output on the host and explain image, container, bind mount, volume, environment variable and Compose.
