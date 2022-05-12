# escape=`
# The MIT License
#
#  Copyright (c) 2019-2020, Alex Earl
#
#  Permission is hereby granted, free of charge, to any person obtaining a copy
#  of this software and associated documentation files (the "Software"), to deal
#  in the Software without restriction, including without limitation the rights
#  to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
#  copies of the Software, and to permit persons to whom the Software is
#  furnished to do so, subject to the following conditions:
#
#  The above copyright notice and this permission notice shall be included in
#  all copies or substantial portions of the Software.
#
#  THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
#  IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
#  FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
#  AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
#  LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
#  OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
#  THE SOFTWARE.

ARG VERSION=4.7-1
FROM jenkins/agent:${VERSION}-windowsservercore-1809

LABEL Description="This is a base image, which allows connecting Jenkins agents via JNLP protocols on Windows" Vendor="Jenkins Project" Version="$VERSION"

ARG user=jenkins

RUN $output = net users ; `
    if(-not ($output -match $env:user)) { `
        Write-Host 'user does not exist?' ; `
        net user $env:user /add /expire:never /passwordreq:no ; `
        net localgroup Administrators /add $env:user ; `
        wmic useraccount WHERE Name=$env:user set PasswordExpires=false; `
    }

####### Update PATH #######
# Env variable with the extra bits for the path
ENV ADD_PATH=";C:\docker-cli;C:\nuget;C:\node;C:\ruby\bin;C:\MSBuildTools\MSBuild\Current\Bin"
# Change the path value directly in the registry key by appending the contents to the env variable to the contents of the path
RUN Set-ItemProperty -Path 'Registry::HKEY_LOCAL_MACHINE\System\CurrentControlSet\Control\Session Manager\Environment' -Name PATH -Value ((Get-ItemProperty -Path 'Registry::HKEY_LOCAL_MACHINE\System\CurrentControlSet\Control\Session Manager\Environment' -Name PATH).path + "$Env:ADD_PATH")
# Verify the path is correct
RUN (Get-ItemProperty -Path 'Registry::HKEY_LOCAL_MACHINE\System\CurrentControlSet\Control\Session Manager\Environment' -Name PATH).path

####### AWS CLI Install #######
ADD https://awscli.amazonaws.com/AWSCLIV2.msi C:/aws-temp/AWSCLIV2.msi
RUN Start-Process -FilePath "C:/aws-temp/AWSCLIV2.msi" -ArgumentList "/qn" -PassThru -Wait | Wait-Process
RUN rd -r "C:/aws-temp"

####### Docker Install #######
# Download and extract docker CLI
ADD https://dockermsft.azureedge.net/dockercontainer/docker-20-10-0.zip C:/docker-temp/docker-binaries.zip
RUN Expand-Archive -Path 'C:/docker-temp/docker-binaries.zip' -DestinationPath 'C:/docker-temp'
RUN mkdir "C:/docker-cli"
RUN move "C:/docker-temp/docker/docker.exe" "C:/docker-cli/"
RUN move "C:/docker-temp/docker/*.dll" "C:/docker-cli/"
RUN rd -r "C:/docker-temp"

####### Docker Compose Install #######
ADD https://github.com/docker/compose/releases/download/1.29.2/docker-compose-Windows-x86_64.exe C:/docker-cli/docker-compose.exe

###### NuGet Install #######
# Install NuGet CLI
ADD https://dist.nuget.org/win-x86-commandline/v5.9.1/nuget.exe C:/nuget/nuget.exe

###### Node Install #######
ADD https://nodejs.org/dist/v16.15.0/node-v16.15.0-win-x64.zip C:/node-temp/node-v16.15.0-win-x64.zip
RUN Expand-Archive -Path 'C:/node-temp/node-v16.15.0-win-x64.zip' -DestinationPath 'C:/node-temp'
RUN move "C:/node-temp/node-v16.15.0-win-x64" "C:/node"
RUN rd -r "C:/node-temp"

####### Ruby Install #######
ADD https://github.com/oneclick/rubyinstaller2/releases/download/RubyInstaller-3.1.2-1/rubyinstaller-devkit-3.1.2-1-x64.exe C:/ruby-temp/rubyinstaller-devkit-3.1.2-1-x64.exe
RUN Start-Process -FilePath "C:/ruby-temp/rubyinstaller-devkit-3.1.2-1-x64.exe" -ArgumentList "/silent","/dir=""C:/ruby""","/tasks=nomodpath" -PassThru -NoNewWindow -Wait | Wait-Process
RUN rd -r "C:/ruby-temp"

###### MSBuild Install #######

# Download the Build Tools bootstrapper.
ADD https://aka.ms/vs/16/release/vs_buildtools.exe C:/msbuild-temp/vs_buildtools.exe
# Restore the default Windows shell for correct batch processing.
SHELL ["cmd", "/S", "/C"]
# Install Build Tools with the Microsoft.VisualStudio.Workload.AzureBuildTools workload, excluding workloads and components with known issues.
RUN C:\msbuild-temp\vs_buildtools.exe --quiet --wait --norestart --nocache `
    --installPath C:\MSBuildTools `
    --add Microsoft.VisualStudio.Workload.AzureBuildTools `
    --add Microsoft.Net.Component.4.8.SDK `
    --add Microsoft.Net.Component.4.5.TargetingPack `
    --remove Microsoft.VisualStudio.Component.Windows10SDK.10240 `
    --remove Microsoft.VisualStudio.Component.Windows10SDK.10586 `
    --remove Microsoft.VisualStudio.Component.Windows10SDK.14393 `
    --remove Microsoft.VisualStudio.Component.Windows81SDK `
 || IF "%ERRORLEVEL%"=="3010" EXIT 0
SHELL ["powershell", "-Command", "$ErrorActionPreference = 'Stop'; $ProgressPreference = 'Continue'; $verbosePreference='Continue';"]
RUN rd -r "C:/msbuild-temp"

# Custom cache invalidation
ARG CACHEBUST=1

####### Compass Install #######
RUN Start-Process -FilePath "C:/ruby/bin/gem.cmd" -ArgumentList "update","--system" -PassThru -NoNewWindow -Wait | Wait-Process
RUN Start-Process -FilePath "C:/ruby/bin/gem.cmd" -ArgumentList "install","compass" -PassThru -NoNewWindow -Wait | Wait-Process
RUN Start-Process -FilePath "C:/ruby/bin/gem.cmd" -ArgumentList "install","bootstrap-sass" -PassThru -NoNewWindow -Wait | Wait-Process

COPY jenkins-agent.ps1 C:/ProgramData/Jenkins
USER ${user}
ENTRYPOINT ["powershell.exe", "-f", "C:/ProgramData/Jenkins/jenkins-agent.ps1"]
