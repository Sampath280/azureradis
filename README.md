```Docker
# Use the appropriate base image
FROM mcr.microsoft.com/dotnet/aspnet:6.0 AS base
WORKDIR /app
EXPOSE 80

FROM mcr.microsoft.com/dotnet/sdk:6.0 AS build
WORKDIR /src
COPY ["Project.Api/Project.Api.csproj", "Project.Api/"]
COPY ["Project.Business/Project.Business.csproj", "Project.Business/"]
RUN dotnet restore "Project.Api/Project.Api.csproj"
COPY . .
WORKDIR "/src/Project.Api"
RUN dotnet build "Project.Api.csproj" -c Release -o /app/build

FROM build AS publish
RUN dotnet publish "Project.Api.csproj" -c Release -o /app/publish

# Copy the XML documentation files
COPY ["src/Project.Api/Project.Api.xml", "src/Project.Business/Project.Business.xml", "/app/publish/"]

FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "Project.Api.dll"]
```
