# escape=`
ARG fromTag=latest

FROM microsoft/windowsservercore:${fromTag}

ARG PS_VERSION=6.1.0-rc.1
ARG POWERSHELL_MSI=https://github.com/PowerShell/PowerShell/releases/download/v${PS_VERSION}/PowerShell-${PS_VERSION}-win-x64.msi
ARG IMAGE_NAME=mcr.microsoft.com/powershell:windowsservercore

LABEL maintainer="PowerShell Team <powershellteam@hotmail.com>" `
      readme.md="https://github.com/PowerShell/PowerShell/blob/master/docker/README.md" `
      description="This Dockerfile will install the latest release of PS." `
      org.label-schema.usage="https://github.com/PowerShell/PowerShell/tree/master/docker#run-the-docker-image-you-built" `
      org.label-schema.url="https://github.com/PowerShell/PowerShell/blob/master/docker/README.md" `
      org.label-schema.vcs-url="https://github.com/PowerShell/PowerShell-Docker" `
      org.label-schema.name="powershell" `
      org.label-schema.vendor="PowerShell" `
      org.label-schema.vcs-ref=${VCS_REF} `
      org.label-schema.version=${PS_VERSION} `
      org.label-schema.schema-version="1.0" `
      org.label-schema.docker.cmd="docker run ${IMAGE_NAME} pwsh -c '$psversiontable'" `
      org.label-schema.docker.cmd.devel="docker run ${IMAGE_NAME}" `
      org.label-schema.docker.cmd.test="docker run ${IMAGE_NAME} pwsh -c Invoke-Pester" `
      org.label-schema.docker.cmd.help="docker run ${IMAGE_NAME} pwsh -c Get-Help"

RUN setx /M PATH "%ProgramFiles%\PowerShell\latest;%PATH%"

# TODO switch to using zip files
# Setup PowerShell - Log-to > C:\Docker.log
SHELL ["C:\\Windows\\System32\\WindowsPowerShell\\v1.0\\powershell.exe", "-command"]
ADD $POWERSHELL_MSI /PowerShell-win-x64.msi

# Install PowerShell package and clean up
RUN $ErrorActionPreference='Stop'; `
    $ConfirmPreference='None'; `
    $VerbosePreference='Continue'; `
    Start-Transcript -path C:\Dockerfile.log -append -IncludeInvocationHeader ; `
    $PSVersionTable | Write-Output ; `
    $VerInfo = Get-ItemProperty -Path 'HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion' ; `
    ('FullBuildString: '+$VerInfo.BuildLabEx) | Write-Output ; `
    ('OperatingSystem:  '+$VerInfo.ProductName+' '+$VerInfo.EditionId+' '+$VerInfo.InstallationType)  | Write-Output ; `
    [System.IO.FileInfo]$MsiFile = Get-Item -Path ./PowerShell-win-x64.msi ; `
    Start-Process -FilePath msiexec.exe -ArgumentList '-qn','-i c:\PowerShell-win-x64.msi', `
      '-log c:\PowerShell-win-x64.msi.log','-norestart' -wait ; `
    $log=get-content -Path C:\PowerShell-win-x64.msi.log -Last 10 ; `
    if ($log -match 'Installation success or error status: 0') { `
      Remove-Item -Path $MsiFile ; `
      $psexe=Get-Item -Path $Env:ProgramFiles\PowerShell\*\pwsh.exe ; `
      New-Item -Type SymbolicLink -Path $Env:ProgramFiles\PowerShell\ -Name latest -Value $psexe.DirectoryName `
    } else { throw 'Installation failed!  See c:\PowerShell-win-x64.msi.log' } ;

# Verify New pwsh.exe runs
SHELL ["C:\\Program Files\\PowerShell\\latest\\pwsh.exe", "-command"]

# Persist %PSCORE% ENV variable for user convenience
ENV PSCORE='"C:\Program Files\PowerShell\latest\pwsh.exe"'

CMD ["pwsh.exe"]
