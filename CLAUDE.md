# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

HomeLab is a containerized personal server application using a microservices architecture. Currently contains a single service (Landing Page) built with ASP.NET Core Blazor Server and Microsoft Fluent UI.

## Technology Stack

- **.NET 10.0** - Latest .NET runtime and SDK
- **ASP.NET Core Blazor** - Interactive Server-Side Rendering (not WebAssembly)
- **Microsoft Fluent UI** - Component library (v4.13.2)
- **Docker** - Multi-stage containerization
- **C#** - With nullable reference types enabled

## Project Structure

The repository follows a microservices pattern:
- `services/` - Container for all microservices
- `services/landing/` - Landing page service
  - `Crews.Internal.HomeLab.Landing/` - Main .NET project
    - `Components/` - Blazor components
      - `Pages/` - Routable page components
      - `Layout/` - Layout wrapper components (MainLayout, NavMenu, ReconnectModal)
    - `wwwroot/` - Static web assets
    - `Program.cs` - Application entry point and service configuration

## Development Commands

### Building and Running

```bash
# Build the project
cd services/landing
dotnet build Crews.Internal.HomeLab.Landing/Crews.Internal.HomeLab.Landing.csproj

# Run the application (local development)
cd services/landing/Crews.Internal.HomeLab.Landing
dotnet run
# Runs on https://localhost:7234 (HTTPS) or http://localhost:5129 (HTTP)

# Build and run with Docker
cd services/landing
docker build -t homelab-landing .
docker run -p 8080:8080 homelab-landing

# Restore NuGet packages
dotnet restore Crews.Internal.HomeLab.Landing/Crews.Internal.HomeLab.Landing.csproj

# Clean build artifacts
dotnet clean Crews.Internal.HomeLab.Landing/Crews.Internal.HomeLab.Landing.csproj
```

### Testing

No test framework is currently configured in this project.

## Architecture Notes

### Blazor Component Model

The application uses **Interactive Server** rendering mode, which means:
- All component logic runs on the server
- UI updates are sent via SignalR WebSocket connection
- Components marked with `@rendermode InteractiveServer` directive
- Connection state handled by `ReconnectModal.razor` component

### Service Registration Pattern

In [Program.cs](services/landing/Crews.Internal.HomeLab.Landing/Program.cs):
1. `AddRazorComponents()` + `AddInteractiveServerComponents()` - Blazor services
2. `AddFluentUIComponents()` - Fluent UI service registration
3. `MapRazorComponents<App>()` + `AddInteractiveServerRenderMode()` - Routing setup

### Error Handling

- Production errors: Redirected to `/Error` page
- 404 errors: Custom handling via `/not-found` route
- Status code pages: Re-execution enabled for custom error pages

### Docker Build Strategy

Multi-stage build separates SDK (build-time) from runtime (deployment):
1. **Build stage**: Uses `dotnet/sdk:10.0` image to compile and publish
2. **Runtime stage**: Uses smaller `dotnet/aspnet:10.0` image for production
3. Container exposes port 8080 (not HTTPS in container)

## Important Configuration

### Project Settings (.csproj)

- `BlazorDisableThrowNavigationException` - Set to `true` to suppress navigation exceptions during render
- Target framework: `net10.0`
- Nullable reference types and implicit usings enabled

### Development Ports

- **HTTP**: 5129
- **HTTPS**: 7234
- **Container**: 8080

### Adding New Pages

1. Create `.razor` file in `Components/Pages/`
2. Add `@page "/route"` directive at the top
3. Optionally add `@rendermode InteractiveServer` for interactivity
4. Update `NavMenu.razor` if navigation link needed

### Adding New Services (Future)

Place new microservices in the `services/` directory following the same structure:
```
services/
└── [service-name]/
    ├── Dockerfile
    └── [Project Files]
```
