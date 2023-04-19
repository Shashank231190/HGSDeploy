# Settings up Variables    

    $GHostPlainPassword="Welcome@1234" #password to access Guarded nodes Compute1 and Compute2
    $HGSPlainPassword   ="Welcome@1234" #password to access HGS cluster nodes. In production environments it should be different

    $SafeModeAdministratorPlainPassword="Welcome@1234" #SafeModePassword for HGS Domain
    $HGSDomainName='Hgslab.local'
    $HGSServiceName = 'MyHGS'

#Create creds

    $GHostPassword = ConvertTo-SecureString $GHostPlainPassword -AsPlainText -Force
    $HGSPassword = ConvertTo-SecureString $HGSPlainPassword -AsPlainText -Force

    $GHostCreds = New-Object System.Management.Automation.PSCredential ("Pikachu\Administrator", $GHostPassword)
    $HGSCreds = New-Object System.Management.Automation.PSCredential ("Administrator", $HGSPassword)
    $HGSDomainCreds = New-Object System.Management.Automation.PSCredential ("$HGSDomainName\Administrator", $HGSPassword)

#wait until machines are up and grab IPs

    do{
        $HGSServerIPs=Invoke-Command -VMName *HGS1, *HGS2 -Credential $HGSCreds -ScriptBlock {(Get-NetIPAddress -InterfaceAlias Ethernet -AddressFamily     IPv4).IPAddress} -ErrorAction SilentlyContinue
        Start-Sleep 5
    }until ($HGSServerIPs.count -eq 3)

# Configuring HGS Server

#Install required HGS feature on HGS VMs

    Invoke-Command -VMName *HGS1,*HGS2 -Credential $HGSCreds -ScriptBlock {
        Install-WindowsFeature -Name HostGuardianServiceRole -IncludeManagementTools
    }

#restart VM's

    Restart-VM -VMName *HGS* -Type Reboot -Force -Wait -For HeartBeat

#Install HGS on first node

    Invoke-Command -VMName *HGS1 -Credential $HGSCreds -scriptblock {
        $SafeModeAdministratorPassword = ConvertTo-SecureString -AsPlainText $using:SafeModeAdministratorPlainPassword -Force
        Install-HgsServer -HgsDomainName $using:HGSDomainName -SafeModeAdministratorPassword $SafeModeAdministratorPassword #-Restart
    }

#restart HGS1

    Restart-VM -VMName *HGS1 -Type Reboot -Force -Wait -For HeartBeat

# Setting up DNS Forwarder

#Set the DNS forwarder on the fabric DC so other nodes can find the new domain

    Invoke-Command -VMName *DC -Credential $FabricCreds -ScriptBlock {
        Add-DnsServerConditionalForwarderZone -Name $using:HGSDomainName -ReplicationScope Forest -MasterServers $using:HgsServerIPs
    }

#wait for DC to be initialized
#Note: Sometimes DC starts for quite some time (Please wait for the Group Policy Client or Applying Computer settings).

    $Result=$null
    do {
        $Result=Invoke-Command -VMName *HGS1 -Credential $HGSDomainCreds -ScriptBlock {
            Get-ADComputer -Filter * -Server HGS1 -ErrorAction SilentlyContinue
            Start-Sleep 5
        }
    }until($Result)

#Wait for HGS2 to finish dcpromo

    $Result=$null
    do {
        $Result=Invoke-Command -VMName *HGS2 -Credential $HGSDomainCreds -ScriptBlock {
            Get-ADComputer -Filter * -Server HGS2
            Start-Sleep 5
        }
    }until($Result)


# Add HGSServer on HGS2 

    Invoke-Command -VMName *HGS2 -Credential $HGSCreds -ScriptBlock {
        $SafeModeAdministratorPassword = ConvertTo-SecureString -AsPlainText $using:SafeModeAdministratorPlainPassword -Force
        Install-HgsServer -HgsDomainName $using:HGSDomainName -HgsDomainCredential $using:HGSDomainCreds -SafeModeAdministratorPassword $SafeModeAdministratorPassword #-Restart
    }

#restart HGS2 

    Restart-VM -VMName *HGS2 -Type Reboot -Force -Wait -For HeartBeat

# Creating Certificates for HGS Server

#you can create CA in Bastion forest https://docs.microsoft.com/en-us/windows-server/security/guarded-fabric-shielded-vm/guarded-fabric-obtain-certs#request-certificates-from-your-certificate-authority

#or just create self signed cert

    Invoke-Command -VMName *HGS1 -Credential $HGSDomainCreds -ScriptBlock {
    $certificatePassword = ConvertTo-SecureString -AsPlainText -String "Welcome@1234" -Force
    $signCert = New-SelfSignedCertificate -Subject "CN=HGS Signing Certificate"
    Export-PfxCertificate -FilePath $env:temp\signCert.pfx -Password $certificatePassword -Cert $signCert
    Remove-Item $signCert.PSPath
    $encCert = New-SelfSignedCertificate -Subject "CN=HGS Encryption Certificate"
    Export-PfxCertificate -FilePath $env:temp\encCert.pfx -Password $certificatePassword -Cert $encCert
    Remove-Item $encCert.PSPath
    Initialize-HgsServer -HgsServiceName $using:HGSServiceName -SigningCertificatePath "$env:temp\signCert.pfx" -SigningCertificatePassword $certificatePassword -EncryptionCertificatePath "$env:Temp\encCert.pfx" -EncryptionCertificatePassword $certificatePassword -TrustTpm -hgsversion V1
    }


# Join HGS2 to the cluster

    Invoke-Command -VMName *HGS2 -Credential $HGSDomainCreds -ScriptBlock {
        Initialize-HgsServer -HgsServerIPAddress $using:HGSServerIPs[0]
    }

# Set HGS configuration to support VMs (disable IOMMU requirement)

    Invoke-Command -VMName *HGS1 -Credential $HGSDomainCreds -ScriptBlock {
        Disable-HgsAttestationPolicy Hgs_IommuEnabled
    }

# Install HostGuardian Hyper-V Support on compute nodes

    Invoke-Command -VMName *Compute1,*Compute2 -Credential $FabricCreds -ScriptBlock {
        Install-WindowsFeature HostGuardian -IncludeManagementTools
    }

#Restart compute nodes

    Restart-VM -VMName *Compute1,*Compute2 -Type Reboot -Force -Wait -For HeartBeat

#Wait for installation to complete
#Start-Sleep 60

#Set registry key to not require IOMMU for VBS in VMs and apply default CI policy
#Also generate attestation artifacts (CI policy, TPM EK, and TPM baseline)
#You should also include https://docs.microsoft.com/en-us/windows/security/threat-protection/windows-defender-application-control/microsoft-recommended-block-rules

# Generating attestation artifacts (CI policy, TPM EK, and TPM baseline)

#grab recommended xml blocklist from GitHub
#[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12

    $content=Invoke-WebRequest -UseBasicParsing -Uri https://raw.githubusercontent.com/MicrosoftDocs/windows-itpro-docs/master/windows/security/threat-protection/windows-defender-application-control/microsoft-recommended-block-rules.md
    #find start and end
    $XMLStart=$content.Content.IndexOf("<?xml version=")
    $XMLEnd=$content.Content.IndexOf("</SiPolicy>")+11 # 11 is lenght of string
    #create xml
    [xml]$XML=$content.Content.Substring($xmlstart,$XMLEnd-$XMLStart) #find XML part

    Invoke-Command -VMName *Compute1, *Compute2 -Credential $FabricCreds -ScriptBlock {
        Set-ItemProperty -Path HKLM:\SYSTEM\CurrentControlSet\Control\DeviceGuard -Name RequirePlatformSecurityFeatures -Value 0
        md C:\attestationdata
        $cipolicy = "C:\attestationdata\CI_POLICY_AUDIT.xml"
        Copy-Item "$env:SystemRoot\schemas\CodeIntegrity\ExamplePolicies\AllowMicrosoft.xml" $cipolicy -Force
        #add recommended XML blocklist
        ($using:XML).Save("$env:TEMP\blocklist.xml")
        #add to MyPolicy.xml
        $mergedPolicyRules = Merge-CIPolicy -PolicyPaths "$env:TEMP\blocklist.xml",$cipolicy -OutputFilePath $cipolicy
        Write-Host ('Merged policy contains {0} rules' -f $mergedPolicyRules.Count)
        # For testing, convert the policy to an audit policy to avoid constrained language mode in PS
        Set-RuleOption -FilePath $cipolicy -Option 3
        # Allowing a CI policy to be updated without a reboot can allow someone to pass attestation and replace with a bad policy, so we disallow that
        Set-RuleOption -FilePath $cipolicy -Option 16 -Delete
        ConvertFrom-CIPolicy -XmlFilePath $cipolicy -BinaryFilePath "C:\attestationdata\CI_POLICY_AUDIT.bin"
        Copy-Item "C:\attestationdata\CI_POLICY_AUDIT.bin" "$env:SystemRoot\System32\CodeIntegrity\SIPolicy.p7b" -Force
        Initialize-Tpm
        (Get-PlatformIdentifier -Name $env:COMPUTERNAME).Save("C:\attestationdata\TPM_EK_$env:COMPUTERNAME.xml")
        Get-HgsAttestationBaselinePolicy -Path "C:\attestationdata\TPM_Baseline_$env:COMPUTERNAME.xml" -SkipValidation
    }

#Reboot VMs again for setting to take effect

    Restart-VM -Name *Compute1,*Compute2 -Type Reboot -Force -Wait -For HeartBeat

# Collect attestation artifacts from hosts

    $HGS1Session = New-PSSession -VMName *HGS1 -Credential $HGSDomainCreds
    $Compute1Session = New-PSSession -VMName *Compute1 -Credential $FabricCreds
    $Compute2Session = New-PSSession -VMName *Compute2 -Credential $FabricCreds

    #Create folder on HGS1
    Invoke-Command -Session $HGS1Session -ScriptBlock {
        New-Item -Name AttestationData -Path c:\ -ItemType Directory
    }

#Copy files

        Copy-Item -Path "C:\attestationdata\TPM_EK_COMPUTE1.xml" -Destination $env:Temp -FromSession $Compute1Session
        Copy-Item -Path "$env:temp\TPM_EK_COMPUTE1.xml" -Destination C:\attestationdata\ -ToSession $HGS1Session
        Copy-Item -Path "C:\attestationdata\TPM_EK_COMPUTE2.xml" -Destination $env:Temp -FromSession $Compute2Session
        Copy-Item -Path "$env:temp\TPM_EK_COMPUTE2.xml" -Destination C:\attestationdata\ -ToSession $HGS1Session
        Copy-Item -Path "C:\attestationdata\TPM_Baseline_COMPUTE1.xml" -Destination $env:Temp -FromSession $Compute1Session
        Copy-Item -Path "$env:temp\TPM_Baseline_COMPUTE1.xml" -Destination C:\attestationdata\ -ToSession $HGS1Session
        Copy-Item -Path "C:\attestationdata\CI_POLICY_AUDIT.bin" -Destination $env:Temp -FromSession $Compute1Session
        Copy-Item -Path "$env:temp\CI_POLICY_AUDIT.bin" -Destination C:\attestationdata\ -ToSession $HGS1Session


# Import the attestation policies on HGS

    Invoke-Command -VMName *HGS1 -Credential $HGSDomainCreds -ScriptBlock {
        # Every individual EK needs to be added
        Add-HgsAttestationTpmHost -Path C:\attestationdata\TPM_EK_COMPUTE1.xml -Force
        Add-HgsAttestationTpmHost -Path C:\attestationdata\TPM_EK_Compute2.xml -Force

# But only one copy of the baseline and CI policy, since they should be identical on both hosts

    Add-HgsAttestationTpmPolicy -Path C:\attestationdata\TPM_Baseline_COMPUTE1.xml -Name "Hyper-V TPM Baseline"
    Add-HgsAttestationCIPolicy -Path C:\attestationdata\CI_POLICY_AUDIT.bin -Name "AllowMicrosoft-AUDIT-CI"
    }

# Now, have the hosts try to attest

    Invoke-Command -VMName *Compute1, *Compute2 -Credential $GHostCreds -ScriptBlock {
    Set-HgsClientConfiguration -AttestationServerUrl "http://$using:HGSServiceName.$using:HGSDomainName/Attestation" -KeyProtectionServerUrl    "http://$using:HGSServiceName.$using:HGSDomainName/KeyProtection"
    }
 
