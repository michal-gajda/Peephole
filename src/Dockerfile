FROM mcr.microsoft.com/dotnet/sdk:7.0 AS build
WORKDIR /src

COPY Peephole.sln Peephole.sln
COPY Peephole.WebApi/Peephole.WebApi.csproj Peephole.WebApi/Peephole.WebApi.csproj
RUN dotnet restore

COPY . .

RUN dotnet build --configuration Debug -o /app/build
RUN dotnet publish --configuration Debug -o /app/publish

FROM mcr.microsoft.com/dotnet/aspnet:7.0
WORKDIR /app

RUN apt-get update && apt-get install -y curl=7.74.0-1.3+deb11u7 --no-install-recommends && rm -rf /var/lib/apt/lists/*
RUN curl -sSL https://aka.ms/getvsdbgsh | bash /dev/stdin -v vs2022 -l /vsdbg

COPY --from=build /app/publish .

RUN groupadd --gid 10000 dotnet && useradd --uid 10000 --gid dotnet --create-home --home-dir /app dotnet && chown -R dotnet:dotnet /app
USER dotnet:dotnet

ENV ASPNETCORE_URLS http://*:5080
EXPOSE 5080

HEALTHCHECK --interval=5s --timeout=1s --retries=3 CMD curl --fail http://localhost:5080/health || exit 1

ARG ASPNETCORE_ENVIRONMENT
ENV ASPNETCORE_ENVIRONMENT $ASPNETCORE_ENVIRONMENT

ENTRYPOINT ["dotnet", "Peephole.WebApi.dll"]
