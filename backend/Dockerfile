FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src

COPY OrbitBoard.Api/OrbitBoard.Api.csproj OrbitBoard.Api/
RUN dotnet restore OrbitBoard.Api/OrbitBoard.Api.csproj

COPY OrbitBoard.Api/ OrbitBoard.Api/
RUN dotnet publish OrbitBoard.Api/OrbitBoard.Api.csproj -c Release -o /app/publish --no-restore

FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS final
WORKDIR /app

COPY --from=build /app/publish .

ENV ASPNETCORE_URLS=http://+:5200
EXPOSE 5200

ENTRYPOINT ["dotnet", "OrbitBoard.Api.dll"]
