## ![Web App](https://img.shields.io/badge/Web%20App-0078D7?style=for-the-badge&logo=googlechrome&logoColor=white) **The ASP.Net Application**

I have a Legacy ASP.Net application which works in old deprecated version 3.5 .Net Framework.
Due to some dependencies the version can't be upgraded. Since the application is a .Net framework application am planning to use a windows system as host for the docker and run windows containers in it.

Also my plan is to create ***Multi Stage*** Build so as to reduce the final image file size and improved security.

Also while discussing with the developers they told that there is no separate build steps they were performing when they are locally setting up the applciation. Instead they do like **open the .sln file drectly in to Visual studio Pro** and run the app in to the IIS using the button in the VS pro itself.

Hence finding the build steps and knowing the build files locations was bit tricky.


### ✅ **Creating the Docker file**
**DockerFile**

````plaintext
# Build stage
FROM mcr.microsoft.com/dotnet/framework/sdk:3.5-windowsservercore-ltsc2019 AS build
WORKDIR /app

# Copy solution and project files
COPY ./source\ code/PMOscar.sln .
COPY ./source\ code/ .

# Build the application
RUN msbuild PMOscar.sln /p:Configuration=Release

# Final stage
FROM mcr.microsoft.com/dotnet/framework/aspnet:3.5-windowsservercore-ltsc2019
WORKDIR /inetpub/wwwroot

# Copy ALL web content files from the source
COPY --from=build /app/ .

# Configure IIS using appcmd instead of PowerShell cmdlets
RUN powershell -Command \
    Import-Module WebAdministration; \
    Remove-WebSite -Name 'Default Web Site'; \
    New-WebSite -Name 'PMOscar' -Port 80 -PhysicalPath 'C:\inetpub\wwwroot'; \
    New-WebAppPool -Name 'PMOscarPool'; \
    Set-ItemProperty 'IIS:\AppPools\PMOscarPool' -Name managedRuntimeVersion -Value 'v2.0'; \
    Set-ItemProperty 'IIS:\AppPools\PMOscarPool' -Name enable32BitAppOnWin64 -Value $true; \
    & $env:windir\system32\inetsrv\appcmd.exe set app 'PMOscar/' /applicationPool:'PMOscarPool'

# Expose port
EXPOSE 80

# Start IIS
ENTRYPOINT ["C:\\ServiceMonitor.exe", "w3svc"]
````

### ✅ **Setting UP the containers **

**Steps**
````Plaintext

docker build -t pmoscar-app .
docker run -d -p 4963:80 --name pmoscar pmoscar-app

````

## 🔥 **Challenges Faced & How I Solved Them**

### ❌ Issue 1: Build image error : Base image and dependencies not found
