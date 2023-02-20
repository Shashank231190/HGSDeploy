# Attestation modes in the Guarded Fabric solution
![image](https://user-images.githubusercontent.com/71546848/220191567-78ab163c-00ae-4fcb-90ff-a61840b8f7d6.png)

The Attestation Service does two things:
1. The Identity Attestation of the host 
The Service makes sure if this is the right host to trust. If the Service is using TPM, then it knows the ekPub it is requesting. If the Service is AD based, then the host needs to be part of a trusted AD Host Group. That is the HOST identity validation.
2. Measurements
The Attestation Service validates how the host is booted, whether the host has the right configuration, and if the configuration is trusted. Once both validations pass, then the Attestation Certificate is signed by the attestation signing key.

Below are the Modes of attestation available in HGS
1. Admin Trusted attestation
2. TPM-trusted attestation (hardware-based)
3. Host key attestation (based on asymmetric key pairs)

# Admin Trusted attestation
![image](https://user-images.githubusercontent.com/71546848/220190750-ff95b1c2-7ed8-4787-8bd5-84191b6e893c.png)

Shielded VMs can only be decrypted and started on hosts that Fabric Admin has designated as guarded hosts.
By adding them to a security group you create in Fabric (not HGS) Active Directory Domain Services, you can identify hosts as guarded (AD DS). The forest of the Host Guardian Service and the fabric AD must establish a trust connection.

AD based attestation uses the group SID and configure that with Attestation service.
The AD attestation is the way to identify servers that will be able to run the Shielded VMs. The security of these servers will have to be based on external processes and solutions provided by the customer.

![image](https://user-images.githubusercontent.com/71546848/220192682-b8f3058e-2b68-4e3e-bea9-b790e442c012.png)

Admin-trusted attestation is deprecated beginning with Windows Server 2019.


# TPM-trusted attestation (hardware-based)

![image](https://user-images.githubusercontent.com/71546848/220192830-5b31ea51-fb33-4148-9405-1692f92fdefc.png)

Host hardware and firmware must include TPM 2.0 and UEFI 2.3.1 with secure boot enabled
Offers the strongest possible protections 

Only hosts that HGS Admin have designated as guarded hosts, and that are running code they have identified as trusted, can start Shielded VMs.
The technologies that help make sure that the hosts are running trusted code are built into the Windows Server operating system, and include Secure Boot, Measured Boot and Code Integrity policies

Here are some of the measurements that the Attestation Service validates: 
Request coming from the EKPub
This is data signed by the TPM, and used to validate that the TPM chip is trusted.
The TCG Log 
A set of events that show how the host was booted; once the TCG log is verified to be valid, then a check is done to see if the content matches any known good policies.

Code Integrity Policy
There is a hash of the CI that gets placed in the TCG log and the Attestation service checks if the hash matches. The CI Policy ensures that you have the right set of drivers running on the Host OS kernel. It also verifies the UEFI parameters, ensures secure boot is enabled, and that no debuggers are attached. Only then does it issue the attestation certificate once the host passes.
Upon Attestation, the host is granted “Attestation Certificate”.
This certificate is used to unlock the VM’s vTPM.

The Get-Platformidentifier creates an XML file that contains the EKPub for the TPM chip. You take that and configure the attestation service by letting it know that this is a known good and authorized host that is trusted.

![image](https://user-images.githubusercontent.com/71546848/220194329-34c5df80-ec46-47c4-8126-fe4f8803e761.png)


TPM based Attestation Measurements performed/requested by AS

    Authorized List of TPMs (EKpub)
    Request is coming from a known trusted host

    TCG Log Integrity Validation
    Replay TCG Log to match TPM PCR measurements

    Code Integrity Policy
    Ensure host is not running any unknown code
    (e.g. malware or debugger)

    Secure Boot
    Make sure Secure Boot is enabled

    Unified Extensible Firmware Interface
    Validate UEFI secure boot parameters


# Host key attestation (based on asymmetric key pairs)

 Intended to support existing host hardware where TPM 2.0 isn't available. Requires fewer configuration steps and is compatible with commonplace server hardware.
 
 Guarded hosts are approved based on possession of the key.
 
 Reference:
 https://learn.microsoft.com/en-us/windows-server/security/guarded-fabric-shielded-vm/guarded-fabric-create-host-key
 
