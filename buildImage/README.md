# Build a container image
We will build a docker container image from a basic Dotnet Blazor Server App and push it to a registry (dockerhub).
## Prerequisites
* Docker desktop (or any OCI image compatible builder like Podman, Containerd etc.) installed
## Create the Blazor Server App
Let use the dotnet cli to create a Blazor Server App in a folder named BlazorServerApp
```csharp
dotnet new blazorserver -o BlazorServerApp
```
Then let navigate to the directory BlazorServerApp
```
cd BlazorServerApp
```` 
## Create a container image
1) Create the .dockerignore file
2) Create the Dockerfile (multi-stage: build and run)
3) Show Container image 
4) Run and connect to container
5) Create and build single stage container (run with SDK)
6) Show image size in images


### Dockerfile for Multi-stage container image (point 2)

```
FROM mcr.microsoft.com/dotnet/sdk:6.0 AS build-env
WORKDIR /app
COPY *.csproj .
RUN dotnet restore *.csproj
COPY . .
RUN dotnet publish *.csproj -c Release -o /app/publish

FROM mcr.microsoft.com/dotnet/aspnet:6.0
WORKDIR /app/publish
EXPOSE 80
EXPOSE 443
COPY --from=build-env  app/publish/ .
ENTRYPOINT ["dotnet", "REPLACE_WITH_NAME.dll"]
```

### Dockerfile for single stage container image (point 5)

```
FROM mcr.microsoft.com/dotnet/sdk:6.0 AS build-env
WORKDIR /app
COPY *.csproj .
RUN dotnet restore *.csproj
COPY . .
RUN dotnet publish *.csproj -c Release -o /app/publish

WORKDIR /app/publish
EXPOSE 80
EXPOSE 443

ENTRYPOINT ["dotnet", "REPLACE_WITH_NAME.dll", "--urls", "http://0.0.0.0"]
```
