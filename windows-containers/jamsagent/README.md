# Description:
Creates an image running the x64 JAMS Agent, handling the setup of a Run As user with the proper Logon as Batch Rights being granted via Carbon.

It is recommended the user and/or password for the Run As user is updated in the dockerfile prior to building the image. This is merely here to serve as an example, or for initial testing purposes within your environment.

# Environment:
Windows Server Core

# Usage:
**Docker Build**
docker build -t jamsagent:latest .

**Docker Run**

This will start the image in a Command Prompt. You can verify the JAMS Service is running by opening PowerShell, and issuing Get-Services *JAMS*.
```
docker run -i jamsagent cmd
```

## Dockerfile Details:
```
# This dockerfile utilizes components licensed by their respective owners/authors.
FROM microsoft/windowsservercore

LABEL Description="JAMSAgent" Vendor="MVP Systems Software" Version="6.5.27"

RUN @powershell wget https://s3-us-west-2.amazonaws.com/devontest/SetupAgentx64.msi -OutFile C:\SetupAgentx64.msi 
RUN msiexec /i "SetupAgentx64.msi" /qn
RUN @powershell Remove-Item C:\SetupAgentx64.msi
RUN net user jamsuser jamspass1! /ADD
RUN @powershell wget https://chocolatey.org/install.ps1 -OutFile C:\chocolatey.ps1
RUN @powershell C:\chocolatey.ps1
RUN @powershell choco install Carbon -y
RUN PowerShell -Command \ 
	Import-Module Carbon; \
	Grant-Privilege -Identity jamsuser -Privilege SeBatchLogonRight; \
	Remove-Item C:\chocolatey.ps1 \
```
