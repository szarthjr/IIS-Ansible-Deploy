# Vagrant-box

A criação da box com o windows 10 seguiu os seguintes passos:

1. Criação de VM no virtual box e instalação do SO
1. Criação de usuário vagrant com password vagrant como administrador
1. Instalação do *VirtualBox Guest Additions*
1. Configuração da VM de acordo com as recomendações do [Vagrant](https://www.vagrantup.com/docs/boxes/base.html#windows-boxes) e para executar o ansible com os seguintes comandos no powerhell/cmd com privilégio de administrador

   ```PowerShell
   # Turn off UAC
   Set-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" -Name "ConsentPromptBehaviorAdmin" -Value "0"
   reg add HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Policies\System /v EnableLUA /d 0 /t REG_DWORD /f /reg:64
   # Disable complex passwords
   secedit /export /cfg c:\secpol.cfg
   (gc C:\secpol.cfg).replace(“PasswordComplexity = 1”, “PasswordComplexity = 0”) | Out-File C:\secpol.cfg
   secedit /configure /db c:\windows\security\local.sdb /cfg c:\secpol.cfg /areas SECURITYPOLICY
   rm -force c:\secpol.cfg -confirm:$false
   # Disable "Shutdown Tracker"
   reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows NT\Reliability" /v ShutDownReasonOn /t REG_DWORD /d 0 /f
   # Base WinRM Configuration
   winrm quickconfig -q
   winrm set winrm/config/winrs '@{MaxMemoryPerShellMB="512"}'
   winrm set winrm/config '@{MaxTimeoutms="1800000"}'
   winrm set winrm/config/service '@{AllowUnencrypted="true"}'
   winrm set winrm/config/service/auth '@{Basic="true"}'
   ```

1. Configuração da VM utilizando o script [./WinRM.ps1](./WinRM.ps1) como Administrador
1. Instalação do IIS, rodar o seguinte comando powershell:

   ```Powershell
   Set-ExecutionPolicy Bypass -Scope Process
   Enable-WindowsOptionalFeature -Online -FeatureName IIS-WebServerRole
   Enable-WindowsOptionalFeature -Online -FeatureName IIS-WebServer
   Enable-WindowsOptionalFeature -Online -FeatureName IIS-CommonHttpFeatures
   Enable-WindowsOptionalFeature -Online -FeatureName IIS-HttpErrors
   Enable-WindowsOptionalFeature -Online -FeatureName IIS-HttpRedirect
   Enable-WindowsOptionalFeature -Online -FeatureName IIS-ApplicationDevelopment

   Enable-WindowsOptionalFeature -online -FeatureName NetFx4Extended-ASPNET45
   Enable-WindowsOptionalFeature -Online -FeatureName IIS-NetFxExtensibility45

   Enable-WindowsOptionalFeature -Online -FeatureName IIS-HealthAndDiagnostics
   Enable-WindowsOptionalFeature -Online -FeatureName IIS-HttpLogging
   Enable-WindowsOptionalFeature -Online -FeatureName IIS-LoggingLibraries
   Enable-WindowsOptionalFeature -Online -FeatureName IIS-RequestMonitor
   Enable-WindowsOptionalFeature -Online -FeatureName IIS-HttpTracing
   Enable-WindowsOptionalFeature -Online -FeatureName IIS-Security
   Enable-WindowsOptionalFeature -Online -FeatureName IIS-RequestFiltering
   Enable-WindowsOptionalFeature -Online -FeatureName IIS-Performance
   Enable-WindowsOptionalFeature -Online -FeatureName IIS-WebServerManagementTools
   Enable-WindowsOptionalFeature -Online -FeatureName IIS-IIS6ManagementCompatibility
   Enable-WindowsOptionalFeature -Online -FeatureName IIS-Metabase
   Enable-WindowsOptionalFeature -Online -FeatureName IIS-ManagementConsole
   Enable-WindowsOptionalFeature -Online -FeatureName IIS-BasicAuthentication
   Enable-WindowsOptionalFeature -Online -FeatureName IIS-WindowsAuthentication
   Enable-WindowsOptionalFeature -Online -FeatureName IIS-StaticContent
   Enable-WindowsOptionalFeature -Online -FeatureName IIS-DefaultDocument
   Enable-WindowsOptionalFeature -Online -FeatureName IIS-WebSockets
   Enable-WindowsOptionalFeature -Online -FeatureName IIS-ApplicationInit
   Enable-WindowsOptionalFeature -Online -FeatureName IIS-ISAPIExtensions
   Enable-WindowsOptionalFeature -Online -FeatureName IIS-ISAPIFilter
   Enable-WindowsOptionalFeature -Online -FeatureName IIS-HttpCompressionStatic
   Enable-WindowsOptionalFeature -Online -FeatureName IIS-ASPNET45
   ```

  obs: Diversas dessas *Features* não serão utilizadas, porém este é um set *default* que poderia ser instalado.

1. Realizar o Pack da Box com o comando:

   ```bash
   vagrant package --base windows10IISAnsible
   ```

1. Realizar o upload da box