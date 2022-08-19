******** Part 3 Deploying Shielded VM ********

Once the HGS service and guarded fabric are in place I can move on to the final step of this test deployment – shielding the existing virtual machine(s). 
The high-level steps for this procedure includes configuring the virtual machine on some other Hyper-V host – 
Such hosts the tenant Hyper-V hosts – exporting and importing it to some guarded host. 
In my network there’re is one guarded host – GuardHost1 – and one tenant 2k19N2. 
So now we going to use tenant 2k19N2 to prepare the VM.

1. VM which needs to prepared to become shielded VM - needs to be turned off.

First off I retreive HGS guardian metadata from the HGS server:

2.Invoke-WebRequest http://hgscluster.hgslab.local/KeyProtection/service/metadata/2014-07/metadata.xml -OutFile C:\HGS\HGSGuardian.xml
Bring this file "HGSGuardian.xml" on the tenant host.

3.Then let’s create the new guardian object that will serve as the VM’s owner using the new self-signed certificates:
Before running the below command make sure Host Guardian Hyper-V support (HGS client service is installed on the tenant)
$OwnerGuardian = New-HgsGuardian –Name ‘OwnerGuardian’ –GenerateCertificates


