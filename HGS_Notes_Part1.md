# ******** Part 1 Creating HGS Server ********

IP address : 80.0.0.1
Name: HGS1

Note : HGS role/service leverage the windows failover cluster, so it is recommended to have more than one node in the setup, however for this lab demonstration, 
I am only using on server 2016 DC edition with latest patches.

1. Installed Windows Server 2016 1607 build.
2. Applied latest updates.
3. Install Host Gaurdian Role

    a. This will install ACtive Directory Role
    #b. It will become member of failover cluster
    c. Automatically configured with a "Just Enough Administration" and registered with pre-defined Active Directory User groups.


4. Subsequent configuration steps will be done in Microsoft PowerShell and the next step is to install HGS Service 
   (The name of the new Active Directory domain will be HGSLab.Local)


<#
This step will setup a domain controller. As we have done this step already - lets move to the next step
Note : Use the powerShell as powerShell_ise may fail to configure the cluster in final stage.
All these proceedings are done under domain admin, in case you are not logged in with domain admin, then 
in that case you have to add the login account to Schema admins default group, as well as other administrative groups
#>
 
    $adminPassword =  ConvertTo-SecureString -AsPlainText "password" -Force
    Install-HgsServer -HgsDomainName ‘HGSLab.Local’ -SafeModeAdministratorPassword $adminPassword -Restart


5. The next step is to create a couple of certificates that will be used by HGS Service for key signing and encryption – I’ll make use of self-signed certificates as it’ll be enough for this installation

<#
Creating Certificates which help in key signing and encryption
Note: Remember the password given at the prompt - I used the password "_abc123"
#>
    
    $certpassword = Read-Host -AsSecureString -Prompt “Enter a password for the PFX file”

    $signCert = New-SelfSignedCertificate -Subject “CN=HGS Signing Certificate”
    Export-PfxCertificate -FilePath signCert.pfx -Password $certpassword -Cert $signCert

    $encCert = New-SelfSignedCertificate -Subject “CN=HGS Encryption Certificate”
    Export-PfxCertificate -FilePath encCert.pfx -Password $certpassword -Cert $encCert

6. Initialze the HGS Server, failover Cluster along with Attestation method which is TrustActiveDirectory.

<#

  Time to initialize the Hgs Server & failover cluster
  Attestation Mode - TrustActiveDirectory
  In step 5 certificates are collected under c:\certs directory

#>

    $signCertPass  = Read-Host -AsSecureString "Signing certificate password"
    $encryptCertPass = Read-Host -AsSecureString "encryption certificate password"
    Initialize-HgsServer -HgsServiceName ‘HgsCluster’ -SigningCertificatePath ‘C:\certs\signCert.pfx’ `
     -SigningCertificatePassword $signCertPass -EncryptionCertificatePath ‘C:\certs\encCert.pfx’ -EncryptionCertificatePassword $encryptCertPass -TrustActiveDirectory

 :Logs will be generated under :  C:\Windows\Logs\HgsServer\
