---
author: Liam Björkman
tags: CI/CD pipleine github actions
---

## The Application

To make the simple "Hello World" ASP.NET Core app run in a container I have simply created a Dockerfile that describes the steps to take to build the app.

Dockerfile:

```dockerfile
FROM mcr.microsoft.com/dotnet/aspnet:3.1-focal AS base
WORKDIR /app
EXPOSE 5000

ENV ASPNETCORE_URLS=http://+:5000 # Set ASP NET Core APP to use port 5000

FROM mcr.microsoft.com/dotnet/sdk:3.1-focal AS build # Build app as Release to directory /app/build
WORKDIR /src
COPY ["SimpleWebHalloWorld.csproj", "./"]
RUN dotnet restore "SimpleWebHalloWorld.csproj"
COPY . .
WORKDIR "/src/."
RUN dotnet build "SimpleWebHalloWorld.csproj" -c Release -o /app/build

FROM build AS publish # Publish the app
RUN dotnet publish "SimpleWebHalloWorld.csproj" -c Release -o /app/publish

FROM base AS final # Run the app
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "SimpleWebHalloWorld.dll"]
```

The image can also be built with the docker-compose file:

```yaml
version: "3.9"
services: 
  web:
    build: . # build the dockerfile in the current directory (root)
    ports:
      - "5000:5000"
```

Description of the dockerfile

First we specify the correct framework for the container to download (3.1 since the app is based on that)

We also set the directory for the files and expose port 5000. Beneath we set the internal ASP NET enviroment to also use port 5000. (This is also the default, but the env command can be used to set other ports if needed.)

After that we build the app as release using dotnet SDK to the src folder. We also publish the app before running it, this is not necessary since we are building the application in the same environment but if it was built outside the container, we would need to publish it to be able to run it.  

GithHub Workflow file, describes a full **Continuous Integration / Continuous Delivery pipeline**.

```yaml
name: DotNet CI/CD

on: [push]

jobs:
    # name: build & test linux for container
    build: # Build & Test, this is the CI part.
        runs-on: ubuntu-latest
        
        steps:
        - uses: actions/checkout@v2
        - uses: actions/setup-dotnet@v1
          with: 
            dotnet-version: 3.1.x
        - name: Build Project
          run: dotnet build
        - name: Run Tests
          run: dotnet test
          
    Build-and-Push-Docker-Image: # This is the CD part, since we are not pushing this to any customers / production environments, CD stands for Continiuous Delivery in this case.
        runs-on: ubuntu-latest
        needs: build
        name: Docker Build, Tag, Push
        env:
          REGISTRY: ghcr.io # Github Container Registry, supersedes GitHUb Docker Registry
        
        steps:
        - name: Checkout repository
          uses: actions/checkout@v2

        - name: Log in to the Container registry
          uses: docker/login-action@v1
          with:
            registry: ${{ env.REGISTRY }} # Example of how names can be set dynamically by the environment
            username: ${{ github.actor }} # Gets our username from the environment it is run in.
            password: ${{ secrets.GITHUB_TOKEN }} # Automatically gets our token from GH.

        - name: Build and push Docker image
          uses: docker/build-push-action@v2 # GH Action that builds the image and pushes it to the specified registry.
          with:
            context: .
            push: true
            tags: ghcr.io/bjork-dev/helloworld:latest # hardcoded, could be replaced with metadata in a more advanced project with more branches
```

Most steps are described with comments above, but there are a few lines which are worth describing more.

```yaml
username: ${{ github.actor }} # Gets our username from the environment it is run in.
password: ${{ secrets.GITHUB_TOKEN }} # Automatically gets our token from GH.
```

Here we automatically get our github username and token secret for authentication against the container registry using GitHub Context. By using special variables we do not have to expose sensitive information in our YAML file. Instead it is resolved at runtime.

```yaml
tags: ghcr.io/bjork-dev/helloworld:latest # hardcoded, could be replaced with metadata in a more advanced project with more branches
```

As described by the comment, we have hardcoded the name and version of the container in the tags section. But this could be changed in a more advanced workflow to have multiple versions and names. One way of automating this is by using the `docker/metadata-action` which allows us to dynamically set name, tag, etc.

Example

```yaml
tags: ${{ steps.meta.outputs.tags }}
labels: ${{ steps.meta.outputs.labels }}
```

Tag events can be defined in the meta action and could be set by branch names or other patterns for example

```yaml
name: Docker meta
        id: meta
        uses: docker/metadata-action@v3
        with:
          images: name/app
          tags: | # Here we set a few patterns to follow
            type=ref,event=branch
            type=ref,event=pr
            type=semver,pattern={{version}}
            type=semver,pattern={{major}}.{{minor}}
```

The pattern above describes the following with examples

| Event          | Ref                        | Docker Tags              |
| -------------- | -------------------------- | ------------------------ |
| `pull_request` | `refs/pull/2/merge`        | `pr-2`                   |
| `push`         | `refs/heads/master`        | `master`                 |
| `push`         | `refs/heads/releases/v1`   | `releases-v1`            |
| `push tag`     | `refs/tags/v1.2.3`         | `1.2.3`, `1.2`, `latest` |
| `push tag`     | `refs/tags/v2.0.8-beta.67` | `2.0.8-beta.67`          |



Sources

(github-context)[https://docs.github.com/en/actions/reference/context-and-expression-syntax-for-github-actions#github-context]

(github token)[https://docs.github.com/en/actions/reference/authentication-in-a-workflow]

(docker metadata)[https://github.com/docker/metadata-action]