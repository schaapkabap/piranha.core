FROM mcr.microsoft.com/dotnet/sdk:7.0-alpine AS prepare-restore-files
ENV PATH="${PATH}:/root/.dotnet/tools"
RUN dotnet tool install --global --no-cache dotnet-subset --version 0.3.2
WORKDIR /source
COPY . .
RUN dotnet subset restore ./examples/MvcWeb/MvcWeb.csproj --root-directory /source --output restore_subset/


FROM mcr.microsoft.com/dotnet/sdk:7.0-alpine AS build-env
ENV DOTNET_URLS=https://+:57796
ENV ASPNETCORE_ENVIRONMENT=Development
WORKDIR /source

# Copy everything
COPY --from=prepare-restore-files /source/restore_subset .
RUN dotnet restore --runtime linux-musl-x64 ./examples/MvcWeb

COPY . .
#Build and publish a release
WORKDIR ./examples/MvcWeb
RUN dotnet publish "MvcWeb.csproj" --runtime linux-musl-x64 --self-contained false -o out --no-restore


FROM mcr.microsoft.com/dotnet/aspnet:6.0-alpine
ENV DOTNET_SYSTEM_GLOBALIZATION_INVARIANT=false

RUN apk add --no-cache icu-libs krb5-libs libgcc libintl libssl1.1 libstdc++ zlib

WORKDIR /source
COPY --from=build-env /source/examples/MvcWeb/out .

EXPOSE 57796

ENTRYPOINT ["dotnet", "MvcWeb.dll"]

