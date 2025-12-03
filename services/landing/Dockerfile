# Build stage
FROM mcr.microsoft.com/dotnet/sdk:10.0 AS build
WORKDIR /src

# Copy project file and restore dependencies
COPY ["Crews.Internal.HomeLab.Landing/Crews.Internal.HomeLab.Landing.csproj", "Crews.Internal.HomeLab.Landing/"]
RUN dotnet restore "Crews.Internal.HomeLab.Landing/Crews.Internal.HomeLab.Landing.csproj"

# Copy the rest of the application code
COPY . .
WORKDIR "/src/Crews.Internal.HomeLab.Landing"

# Build and publish the application
RUN dotnet build "Crews.Internal.HomeLab.Landing.csproj" -c Release -o /app/build --no-restore
RUN dotnet publish "Crews.Internal.HomeLab.Landing.csproj" -c Release -o /app/publish --no-build /p:UseAppHost=false

# Runtime stage
FROM mcr.microsoft.com/dotnet/aspnet:10.0 AS runtime
WORKDIR /app

# Copy published application from build stage
COPY --from=build /app/publish .

# Expose port 8080 (default for ASP.NET Core)
EXPOSE 8080

# Set environment variable for ASP.NET Core
ENV ASPNETCORE_URLS=http://+:8080

# Run the application
ENTRYPOINT ["dotnet", "Crews.Internal.HomeLab.Landing.dll"]
