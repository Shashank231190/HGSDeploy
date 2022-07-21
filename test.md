# ******** Part 2 Configuring Guarded Host ********

  1. Install windows server 2016, 1607 build
  2. Install latest updates
  3. Join gaurded host (call it gaurdhost1)to some other domain controller in the fabric other than HGS Domain Controller.
  4. Install the guarded host & install Hyper-V, HostGuardian (Hyper-V) support - I am using Nested Virtualization
  Install-WindowsFeature Hyper-V, HostGuardian -IncludeManagementTools -Restart

  Name : gaurdhost1
  IP Address : 80.0.0.10
  Logon Domain Controller : Pikachu.Local (Non-HGS)


Before we proceed to configuring guarded hosts there are several steps that should be taken.

1. For guarded hosts to be able to retrieve information from the HGS server and vice versa I’ll have to create the DNS forwarders in both domains. 
  Before I do that I’ll create the reverse lookup zones on both domain controllers:

   a. One Domain Controller Promoted when HGS role is installed named as HGSLab.Local domain.  
   b. Other Domain Controller to which gaurdeded host is using as a logon server.

  Once Conditional forwarder is set make sure FQDN of both the DC is pingable to & fro.

2. The one-way forest trust between the hgs and fabric domains must be configured (HGS domain trusts FABRIC domain).

3. Once the one way trust has been created, we need to create a security group on Pikachu.Local Domain & register this on HGS server. This security group will host       the computer accounts which will deploye as GuardedHost and intended to run shielded VM's. Follow the video tutorial to learn about it.


4. Configure the host's key protection & attestation URL's by issuing the following command on guarded host, however to get the URL's jump to HGS server and run
   Get-HGSServer command
   
       PS C:\> get-HGSserver
        Name                           Value
        ----                           -----
        AttestationOperationMode       AD
        AttestationUrl                 {http://hgscluster.hgslab.local/Attestation}
        KeyProtectionUrl               {http://hgscluster.hgslab.local/KeyProtection} 

#

    Set-HgsClientConfiguration -AttestationServerUrl "http://hgscluster.hgslab.local/Attestation" -KeyProtectionServerUrl "http://hgscluster.hgslab.local/KeyProtection"

    Output : 

    IsHostGuarded            : False
    Mode                     : HostGuardianService
    KeyProtectionServerUrl   : http://hgscluster.hgslab.local/KeyProtection
    AttestationServerUrl     : http://hgscluster.hgslab.local/Attestation
    AttestationOperationMode : ActiveDirectory
    AttestationStatus        : InsecureHostConfiguration
    AttestationSubstatus     : VirtualSecureMode

  Note: Current Configuration is still showing up as 
  IsHostGuarded : false
  AttestationStatus : InsecureHostConfiguration

  We need to fix it -
  Ran below command and see the error message as below.

    get-hgsTrace -RunDiagnostics -detailed
    Overall Result: Fail
        GuardHost1: Fail
            Test Attestation: Fail
                Check Attestation Status: Fail
                >>> The host cannot attest because virtual secure mode could not be detected.  Please ensure that the
                >>> Isolated User Mode feature is installed and enabled, and that this is a physical machine (not a virtual
                >>> machine).


  Reason for this failure - I am not using physical host for guarded host, rather using a nested Virtualization and for this to work I need to enable device guard.
  For that we can read from here:
  https://docs.microsoft.com/en-us/windows/win32/procthread/isolated-user-mode--ium--processes

  To enable it on server edition we need to enable certain registry settings.

  - Registry Settings - copy these settings in a bat file and run it administrator

        reg add "HKLM\SYSTEM\CurrentControlSet\Control\DeviceGuard" /v "EnableVirtualizationBasedSecurity" /t REG_DWORD /d 1 /f
        reg add "HKLM\SYSTEM\CurrentControlSet\Control\DeviceGuard" /v "RequirePlatformSecurityFeatures" /t REG_DWORD /d 1 /f
        reg add "HKLM\SYSTEM\CurrentControlSet\Control\DeviceGuard" /v "Locked" /t REG_DWORD /d 0 /f
        reg add "HKLM\SYSTEM\CurrentControlSet\Control\DeviceGuard\Scenarios\HypervisorEnforcedCodeIntegrity" /v "Enabled" /t REG_DWORD /d 1 /f
        reg add "HKLM\SYSTEM\CurrentControlSet\Control\DeviceGuard\Scenarios\HypervisorEnforcedCodeIntegrity" /v "Locked" /t REG_DWORD /d 0 /f

   After enabling the registry and quick reboot, we see attestation test marked as pass.

     PS C:\> get-hgstrace -RunDiagnostics -detailed
     Overall Result: Warning
         GuardHost1: Warning
             Test Attestation: Pass
                 Check Attestation Status: Pass


   Further confirming with below command:

     PS C:\> get-HGSClientConfiguration


     IsHostGuarded            : True
     Mode                     : HostGuardianService
     KeyProtectionServerUrl   : http://hgscluster.hgslab.local/KeyProtection
     AttestationServerUrl     : http://hgscluster.hgslab.local/Attestation
     AttestationOperationMode : ActiveDirectory
     AttestationStatus        : Passed
     AttestationSubstatus     : NoInformation


