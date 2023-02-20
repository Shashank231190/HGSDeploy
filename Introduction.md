# Introduction to HGS

With the evolution of servers to virtual environments, this brings new challenges in trying to protect the datacenter environment.

In addition to the regular attacks already discussed in previous slides, we also have the virtualization fabric that needs to be protected. The virtualization fabric bring new items to be considered when planning for a security strategy:
Administrators have the “keys to the kingdom”, and in the case of sensitive workloads such as Domain Controllers, virtualization admins can access the secrets inside virtual machines. A compromised fabric administrator can be a malicious administrator or an administrative account that was compromised by an attacker.
A virtual machine is really just a file and virtual disks, that can be copied to a USB stick or a laptop and then be mounted in another environment. 
In the past, to protect a server or a sensitive workload, we use to put these servers in a highly secured physical environment that only allowed people would be able to access and have physical access to the assets. In the case of a virtual machine, anyone who have access to the virtualization host will have access to the virtual machine source files, which bring us back to the first statement.
In the last few years, great new capabilities came up such as TPM chips, Secure Boot, UEFI 2.0 and others. The problem is that these features are tied to hardware capabilities that are not exposed to virtual machines.



![image](https://user-images.githubusercontent.com/71546848/220169455-70f0eab6-660c-4407-bda6-94d78ab24a59.png)

What do these attacks have in common?
1. Stolen admin credentials
2. Phishing attacks
3. Pass-the-hash (PtH) attacks
4. Insider attacks
5. Fabric attacks


![image](https://user-images.githubusercontent.com/71546848/220170897-dbcd87d0-367c-45f9-89ae-3bb5900c8f69.png)


