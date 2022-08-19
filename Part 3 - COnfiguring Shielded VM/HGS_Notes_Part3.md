##### Part 3 Deploying Shielded VM 

#####  Once the HGS service and guarded fabric are in place I can move on to the final step of this test deployment – shielding the existing virtual machine(s). 
#####  The high-level steps for this procedure includes configuring the virtual machine on some other Hyper-V host –  called tenant Hyper-V hosts.

#####  Current Setup:

1. Gaurded Host
2. HGS server
3. Using tenant server to prepare shielded VM.

##### Steps: 
##### Turn off the VM which needs to be shielded. First off I retreive HGS guardian metadata from the HGS server:
#####  Note: Create HGS directory on c:\ path on HGS server and then run below command on HGS server.

`Invoke-WebRequest http://hgscluster.hgslab.local/KeyProtection/service/metadata/2014-07/metadata.xml -OutFile C:\HGS\HGSGuardian.xml`

***
Bring this file "HGSGuardian.xml" on the tenant host.
***

##### Then let’s create the new guardian object that will serve as the VM’s owner using the new self-signed certificates:
***
Before running the below command make sure "Host Guardian Hyper-V support" (HGS client service is installed on the tenant)
Note: Guardian Object cannot be created without HGS service module and thats why we need to install this service on tenant host.
***

`$OwnerGuardian = New-HgsGuardian –Name "OwnerGuardian" –GenerateCertificates`

***
Remember this certificate is generated on tenant Host and if user uses this certificate as a protector, than this needs to be imported to the guarded host. This further explains the step 1, where metadata for HGS server will be use to further wrap this ceritificate itself and then VM can be imported without importing the
certificate.
***


##### Create one more guardian to wrap the $OwnerGuardian as a key Protector.

`$Guardian = Import-HgsGuardian -Path "C:\HGS\HGSGuardian.xml" -Name "FabricGuardian" –AllowUntrustedRoot`




> Output for both the objects

> PS C:\> $OwnerGuardian

> Name          HasPrivateSigningKey Signing Certificate Subject
> ----          -------------------- ---------------------------
> OwnerGuardian True                 CN=Shielded VM Signing Certificate (OwnerGuardian) (2k16N1)


> PS C:\> $Guardian

> Name           HasPrivateSigningKey Signing Certificate Subject
> ----           -------------------- ---------------------------
> FabricGuardian False                CN=HGS Signing Certificate

***


##### Create a key protector

`$KeyProtector = New-HgsKeyProtector -Owner $OwnerGuardian -Guardian $Guardian -AllowUntrustedRoot`


##### Configure a key protector for the virtual machine named GuardedVMTest in my lab

`Set-VMKeyProtector -Vmname GuardedVMTest -KeyProtector $KeyProtector.RawData`

##### Set the security policy for the VM GuardedVMTest

`Set-VMSecurityPolicy -VMName GuardedVMTest -shielded $false`

##### Enable vTPM on the GuardedVMTest to be able to use it for BitLocker 

***
Note : This option is only avaiable on gen 2 VM
***

`Enable-VMTPM -VMNAME GuardedVMTest`

***
Note:
Before you move a shielded VM to the guarded host it must prepared for the remote management (WSMan, RDP)! In this test I used the -Shielded $false vm security policy that means fabric administrators still can access the GuardedVMTest virtual machine because it has been shielded using Encryption Supported mode which permits Hyper-V console connections to the shielded VMs. If I had used -Shielded $true vm security policy the only way to connect to the DC vm would be via RDP or WSman.
***

##### Export the GuardedVMTest vm from HV1 Hyper-V host and import it to GuardHost1 which is one of guarded hosts.
