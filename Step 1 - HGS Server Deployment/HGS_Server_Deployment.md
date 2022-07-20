# HGS_Server_Deployment

First we will setup the HGS Server 

# Prerequisites

Hardware: HGS can be run on physical or virtual machines, but physical machines are recommended.

If you want to run HGS as a three-node physical cluster (for availability), you must have three physical servers. (As a best practice for clustering, the three servers should have very similar hardware.)

Operating system: Host key attestation requires Windows Server 2019 Standard or Datacenter edition operating with v2 attestation. For TPM-based attestation, HGS can run Windows Server 2019 or Windows Server 2016, Standard or Datacenter edition.

Server Roles: Host Guardian Service and supporting server roles.

Configuration permissions/privileges for the fabric (host) domain: You will need to configure DNS forwarding between the fabric (host) domain and the HGS domain.

# Steps
Enable Host Guardian Service role by opening windows PowerShell in a elevated mode and run the following command.

Install-WindowsFeature -Name HostGuardianServiceRole -IncludeManagementTools -Restart
 
Install HGS Domain in its own forest by running the below command.

$SafeModeAdministratorPassword= ConvertTo-SecureString -AsPlainText '<password>' -Force 
Note : Replace <Password> with HGS machine password.

Install-HgsServer -HgsDomainName ‘HGSLab.Local' -SafeModeAdministratorPassword $SafeModeAdministratorPassword -Restart
 
Note : Replace ‘HGSLab.local’ with a FQDN of your choice.
 
After machine reboot, log in with the domain account with the same password which you have used for the local account.
# Installing HGS Server will pramote the HGS Server as Domain controller with the domain name provided with HgsDomainName switch.

We can install HGS on second node by following below commands.

$SafeModeAdministratorPassword= ConvertTo-SecureString -AsPlainText '<password>' -Force 
Note : Replace <Password> with HGS machine password.

Install-HgsServer -HgsDomainName ‘HGSLab.Local' -SafeModeAdministratorPassword $SafeModeAdministratorPassword -Restart

