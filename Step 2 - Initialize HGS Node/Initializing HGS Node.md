# Initializing HGS Cluster and Generating Certificates

For initializing HGS Node, administrator need to have a valid SSL certificate. For a lab environment, we can use self-signed certificate. But for production use, has to purchase SSL certificate from digital certificate Vendors.
#you can create CA in Bastion forest https://docs.microsoft.com/en-us/windows-server/security/guarded-fabric-shielded-vm/guarded-fabric-obtain-certs#request-certificates-from-your-certificate-authority

# Generating Self sign Certificates
$CertificatePassword = ConvertTo-SecureString -AsPlainText '<password>' -Force
Note : Replace <Password> with HGS machine password.
$certificatePassword = ConvertTo-SecureString -AsPlainText -String "Password@123" -Force
 
$signCert = New-SelfSignedCertificate -Subject "CN=HGS Signing Certificate"
Export-PfxCertificate -FilePath $env:temp\signCert.pfx -Password $certificatePassword -Cert $signCert

Remove-Item $signCert.PSPath
 
$encCert = New-SelfSignedCertificate -Subject "CN=HGS Encryption Certificate"
Export-PfxCertificate -FilePath $env:temp\encCert.pfx -Password $certificatePassword -Cert $encCert

Remove-Item $encCert.PSPath

# Initializing the HGS Server
  
Initialize-HgsServer -HgsServiceName ‘MyHGS’ -SigningCertificatePath "C:\signCert.pfx" -SigningCertificatePassword $certificatePassword -EncryptionCertificatePath " C:\encCert.pfx" -EncryptionCertificatePassword $certificatePassword -TrustTpm -hgsversion V1

We can add the second node in 
  
Initialize-HgsServer -HgsServerIPAddress 192.168.1.50 <HGS1 Server IP>

  
# Set the DNS forwarder on the Guarded Domain DC so Compute nodes can find the new domain
https://docs.microsoft.com/en-us/windows-server/security/guarded-fabric-shielded-vm/guarded-fabric-configuring-fabric-dns 

Add-DnsServerConditionalForwarderZone -Name HGSLab.local  -ReplicationScope Forest -MasterServers 192.168.1.50
  
Here Guarded domain fqdn is HGSLab.local   with IP 192.168.1.50
  
To add the HGSLab.local   to the trusted group, run the below command.
  
netdom trust HGSLab.local   /domain: HGSLab.local   /userD: HGSLab.local\Administrator /passwordD:<PASSWORD> /add
  
Note : Replace “<PASSWORD>” with appropriate credential details.
  
That’s it, you all done with HGS Server configuration.

