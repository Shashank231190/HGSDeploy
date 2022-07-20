# Live Migration of vTPM enabled VM

When we enable the vTPM on any virtual machine it will create a Key Protector for that VM and that Key protector will be bound with 2 certificates (Signing and Encryption) saved in “Shielded VM Local Certificates”.

![image](https://user-images.githubusercontent.com/71546848/180096652-946d695f-2a9d-466c-b5ef-7c4963a92671.png)

If these certificates are missing from the destination Host then VM will fail to Live migrate.

# We can see below errors 

Live migration of 'Virtual Machine 'VMname**' failed.

Virtual machine migration operation for ''VMname**'’ failed at migration destination 'HostName-**'. (Virtual machine ID <GUID>)
The version of the device 'Microsoft Virtual TPM Device' of the virtual machine 'VMname**' is not compatible with device on physical computer 'HostName-**'. (Virtual machine ID <GUID>)
The key protector for the virtual machine '' could not be unwrapped. HostGuardianService returned: One or more arguments are invalid (0x80070057) . Details are included in the HostGuardianService-Client event log. (Virtual machine ID )

 

Export the Signing and Encryption certificate with private Key from “Shielded VM Local Certificates” folder from the Hyper-V host where we have created the VM with vTPM feature.

 

Import both the Signing and Encryption certificate on the Host where you want the VM to be migrated.

C:\certs>certutil -importPFX "Shielded VM Local Certificates" "C:\Certs\ankit-VMEncryption.pfx"

If the certificates are present on the server, then VM will search for its key protector and VM will find the protector and it will boot up without any issue.



![image](https://user-images.githubusercontent.com/71546848/180096913-48ac6ee1-ed98-4824-98c2-144fb18c65a1.png)
