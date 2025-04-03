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
COPY ./source\ code/asp-app.sln .
COPY ./source\ code/ .

# Build the application
RUN msbuild asp-app.sln /p:Configuration=Release

# Final stage
FROM mcr.microsoft.com/dotnet/framework/aspnet:3.5-windowsservercore-ltsc2019
WORKDIR /inetpub/wwwroot

# Copy ALL web content files from the source
COPY --from=build /app/ .

# Configure IIS using appcmd instead of PowerShell cmdlets
RUN powershell -Command \
    Import-Module WebAdministration; \
    Remove-WebSite -Name 'Default Web Site'; \
    New-WebSite -Name 'asp-app' -Port 80 -PhysicalPath 'C:\inetpub\wwwroot'; \
    New-WebAppPool -Name 'asp-appPool'; \
    Set-ItemProperty 'IIS:\AppPools\asp-appPool' -Name managedRuntimeVersion -Value 'v2.0'; \
    Set-ItemProperty 'IIS:\AppPools\asp-appPool' -Name enable32BitAppOnWin64 -Value $true; \
    & $env:windir\system32\inetsrv\appcmd.exe set app 'asp-app/' /applicationPool:'asp-appPool'

# Expose port
EXPOSE 80

# Start IIS
ENTRYPOINT ["C:\\ServiceMonitor.exe", "w3svc"]
````

### ✅ **Setting UP the containers **

**Steps**
````Plaintext

docker build -t asp-app .
docker run -d -p 4963:80 --name asp-app asp-app

````

## 🔥 **Challenges Faced & How I Solved Them**

### ❌ Issue 1: Build image error : Build files generating location issue

When i was creating the image there is actually two steps in the Docker File. A build stage and Final Stage

After the build is finished in the build stage the files from the directory /app/bin/Release is copied to the Stage build step and paste into Working directory /inetpub/wwwroot

But this throwed an **error**

````plaintext
Step 7/11 : WORKDIR /inetpub/wwwroot
 ---> Using cache
 ---> fae74b049082
Step 8/11 : COPY --from=build /app/bin/Release/ .
COPY failed: stat app\bin\Release\: file does not exist

````

I debugged by the same by building only the build stage (omiting final stage) and when the container is UP manually checking inside that container to see where are the build files located

**Debugging Steps**
````
docker build --target build -t asp-app-build-test .
docker run --rm asp-app-build-test powershell -Command "Get-ChildItem -Recurse -Directory | Where-Object { $_.Name -eq 'bin' }"
docker run --rm asp-app-build-test cmd /c "dir /s /b bin"

````

By doing the same i found out the build files are located in /app/bin. Then i updated the docker file to copy the files from /app/bin to current working directory of Final stage and the build was successful.

### ❌ Issue 2: Container running error : Applicaition is not starting, showing w3svc error

After the image is built i have tried to run the container but it is not starting and exiting automatically.

I have checked using cmd # docker logs asp-app and found the following error

````
Service 'w3svc' has been stopped 
APPCMD failed with error code 4312
Failed to update IIS configuration

````

It looks like the w3svc is not starting becaus some IIS configuration error. 

I have a gut  feeling that the files copied in to the Final stage are not complete since the container is running with that files that is why the service is not running.

So i planned to login to the container and check the files status, for that i have edited the Docker file first so that the container will not start any services so there will not be any issues and i can directly login to the container.

**DockerFile**
````
# Comment out the original ENTRYPOINT
# ENTRYPOINT ["powershell", "-Command", "$ErrorActionPreference = 'Stop'; Start-Service W3SVC; Write-Host 'IIS Started Successfully'; while ($true) { Start-Sleep -Seconds 3600 }"]

# Use this to keep the container running without starting IIS
ENTRYPOINT ["powershell", "-Command", "while ($true) { Start-Sleep -Seconds 3600 }"]
````

**Steps of Debugging**
````
docker build -t asp-app-debug .
docker run -d -p 4963:80 --name asp-app asp-app-debug
docker exec -it asp-app-debug powershell

````
When browsing through the files in the container i have found that the files in the working directory that is C:\inetpub\wwwroot only has the compiled DLLs and no ASP.NET web content files (like .aspx pages, web.config, CSS, JavaScript, images, etc.).

So i figured i need to copy all the web content files from the source directory to the final container, not just the compiled binaries. The application can't run without the web.config and other content files.
