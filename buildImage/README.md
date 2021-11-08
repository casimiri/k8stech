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
