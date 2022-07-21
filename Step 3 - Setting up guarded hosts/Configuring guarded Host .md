# Setting up Guarded Host with TPM attestation
Guarded hosts using TPM mode must meet the following prerequisites:
  Hardware: One host is required for initial deployment. To test Hyper-V live migration for shielded VMs, you must have at least two hosts.
  Hosts must have:
    IOMMU and Second Level Address Translation (SLAT)
    TPM 2.0
    UEFI 2.3.1 or later
    Configured to boot using UEFI (not BIOS or "legacy" mode)
    Secure boot enabled
  Operating system: Windows Server 2016 Datacenter edition or later
 
Important
Make sure you install the latest cumulative update.
Role and features: Hyper-V role and the Host Guardian Hyper-V Support feature. The Host Guardian Hyper-V Support feature is only available on Datacenter editions of Windows Server.

* Add the Guarded Host into Pikachu.Local domain.
Install Host Guardian Feature on Guarded Hosts by using below Powershell.
Install-WindowsFeature HostGuardian -IncludeManagementTools
* Also generate attestation artifacts (CI policy, TPM EK, and TPM baseline)

TPM mode uses a TPM identifier (also called a platform identifier or endorsement key [EKpub]) to begin determining whether a particular host is authorized as "guarded." This mode of attestation uses Secure Boot and code integrity measurements to ensure that a given Hyper-V host is in a healthy state and is running only trusted code. In order for attestation to understand what is and is not healthy, you must capture the following artifacts:
* TPM identifier (EKpub) - 
This information is unique to each Hyper-V host
* TPM baseline (boot measurements) - 
This is applicable to all Hyper-V hosts that run on the same class of hardware
* Code integrity policy (an allowlist of allowed binaries) - 
This is applicable to all Hyper-V hosts that share common hardware and software

It is recommended to capture the baseline and CI policy from a "reference host" that is representative of each unique class of Hyper-V hardware configuration within your datacenter. Beginning with Windows Server version 1709, sample CI policies are included at C:\Windows\schemas\CodeIntegrity\ExamplePolicies.
https://docs.microsoft.com/en-us/windows-server/security/guarded-fabric-shielded-vm/guarded-fabric-tpm-trusted-attestation-capturing-hardware

Endorsement key for each guarded host needs to be added on HGS Sever, But only one copy of the baseline and CI policy, since they should be identical on both hosts
Add-HgsAttestationTpmHost -Path C:\attestationdata\TPM_EK_COMPUTE1.xml -Force
Add-HgsAttestationTpmHost -Path C:\attestationdata\TPM_EK_Compute2.xml -Force
Add TPM Baseline of a Guarded Host on HGS Server
Add-HgsAttestationTpmPolicy -Path C:\attestationdata\TPM_Baseline_COMPUTE1.xml -Name "Hyper-V TPM Baseline1"
Add CI Policy of Guarded Host on HGS Server
Add-HgsAttestationCIPolicy -Path C:\attestationdata\CI_POLICY_AUDIT.bin -Name "AllowMicrosoft-AUDIT-CI1"

Once added we can check the host configuration by running below command.

Get-HgsClientConfiguration

![Hgsclientconfig](https://user-images.githubusercontent.com/71546848/179978085-7fa77b0e-2b68-4d71-99f1-4960b4bb237f.jpg)



Get-HgsTrace -RunDiagnostics -Detailed
