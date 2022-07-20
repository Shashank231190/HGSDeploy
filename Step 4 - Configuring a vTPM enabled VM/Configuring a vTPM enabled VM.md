# Configuring a vTPM enabled VM

A vTPM is a software-based representation of a physical Trusted Platform Module 2.0 chip. A vTPM acts as any other virtual device. You can add a vTPM to a virtual machine in the same way you add virtual CPUs, memory, disk controllers, or network controllers. A vTPM does not require a hardware Trusted Platform Module chip.

If you have a guarded Host and you don't want to use Shielded VM but you need TPM on Virtual Machine then you need to make sure you are running as Local Mode on Guarded Host.
We can switch the mode to Local by using below command.
Set-HGSClientConfiguration -EnableLocalMode

We can check the current client configuration of guarded host by running below command,
Get-HgsClientConfiguration

![image](https://user-images.githubusercontent.com/71546848/180092639-7f3538b6-e6e4-4289-99c7-e2c68a4d42f3.png)


We can switch from Local mode to HGS mode by running below command,
Set-HgsClientConfiguration -AttestationServerUrl "http://Myhgs.Hgslab.local/Attestation" -KeyProtectionServerUrl "http://Myhgs.Hgslab.local/keyProtection"

![image](https://user-images.githubusercontent.com/71546848/180093166-1485c8e3-f6ba-45fd-9bf2-0709911a9de0.png)


When we will enable the vTPM on Virtual Machine it will generate two certificates in certificate store under "Shielded VM Local Certificates"

We can run below commands to enable vTPM on Virtual machine.

    $vm = get-vm -name “VM5"

    Set-VMKeyProtector -VMName “Vm5" -NewLocalKeyProtector

    Enable-VMTPM -VM $vm

    Get-VMSecurity $vm

Each VM will have its own key protector associated with the Signing and Encryption certificate stored in certificate store.

![image](https://user-images.githubusercontent.com/71546848/180095327-5ab681cf-8ec1-476c-aa78-abd125629a1c.png)

After enabling vTPM on Virtual Machine we can check the VM security and find that TPM is enabled.

![image](https://user-images.githubusercontent.com/71546848/180092150-7a74c27e-a814-4ed6-b7b8-e2bcde009794.png)

We can check TPM status inside Vm by running Get-TPM command.

![image](https://user-images.githubusercontent.com/71546848/180091217-944e10ec-3e79-42ee-93d0-94131807acea.png)
