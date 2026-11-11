# Day 66 Interview Guide — Docker for Data Engineering

## 1. What is Docker?
Docker is a platform/toolset for building, distributing and running applications in containers. It helps package an application with runtime dependencies.

## 2. What is a container?
An isolated process environment that runs an application with required files, libraries and configuration while sharing the host operating system kernel.

## 3. What is a Docker image?
A packaged template used to create containers. It includes application files and required runtime layers.

## 4. Image versus container?
An image is the template; a container is an instance created from it. One image can create multiple containers.

## 5. What is a Dockerfile?
A text file containing instructions to build an image, such as FROM, WORKDIR, COPY, RUN, ENV, CMD and ENTRYPOINT.

## 6. What does FROM do?
Selects the base image, such as a Python runtime image.

## 7. What does WORKDIR do?
Sets the working directory inside the image/container for subsequent instructions and default command context.

## 8. COPY versus RUN?
COPY transfers files from build context into the image. RUN executes a command during image build, such as installing packages.

## 9. CMD versus ENTRYPOINT?
CMD provides a default command or arguments. ENTRYPOINT configures the main executable. They can be combined; behavior depends on their forms.

## 10. What is a bind mount?
It maps a specific host filesystem path into a container. It is useful for sharing local input/output folders during development.

## 11. What is a Docker volume?
Persistent storage managed by Docker. It can outlive an individual container.

## 12. Why should data not be stored only inside a container?
A container's writable layer is tied to its lifecycle and may be removed. Persistent business data should live in mounted storage or an external data platform.

## 13. What is port mapping?
It maps a host port to a port inside a container. A batch ETL script without a network listener does not need port publishing.

## 14. What is Docker Compose?
A tool for defining and running multiple related containers from YAML. It can define services, networks, volumes and configuration.

## 15. Why pin Python dependencies?
Pinning improves repeatability and reduces unexpected package upgrades. Production also needs patch management and vulnerability review.

## 16. Why not put secrets in a Dockerfile?
Secrets may be embedded in image layers and exposed to image users. Use runtime secret injection or an approved secret manager.

## 17. Why rebuild after changing app.py?
The application file was copied into the image at build time. Build a new image unless a development bind mount is intentionally used.

## 18. Scenario: container cannot find orders.csv.
Inspect container path, bind mount, host working directory, file name and Docker Desktop file access. Verify the file exists inside the container.

## 19. Scenario: output disappears after container exits.
The application may have written to the container's internal filesystem. Mount a host output folder or named volume so data persists.

## 20. Scenario: image works locally but not in CI.
Compare Docker versions, build context, base image availability, dependency resolution, environment variables, paths, architecture and permissions. Inspect build and container logs.

## 21. Explain your Day 66 project.
I created a Python ETL script, pinned its dependency, wrote a Dockerfile, built an image and ran it with input/output bind mounts. I verified city totals in a host output CSV and added a Compose definition. I documented failure handling and checked that no secrets were embedded in the image.

## Common interview traps
- Confusing image and container.
- Assuming containers are full virtual machines.
- Storing output only in an ephemeral container layer.
- Baking passwords into images.
- Forgetting to rebuild after copying changed code.
- Publishing ports for a batch script that does not listen.
- Assuming Docker automatically makes an application secure.

## Readiness checklist
- [ ] Explain image, container and Dockerfile.
- [ ] Explain Dockerfile instructions.
- [ ] Explain bind mounts and named volumes.
- [ ] Explain port mapping and Compose.
- [ ] Describe build/run/log troubleshooting.
- [ ] Explain dependency pinning and image security.
