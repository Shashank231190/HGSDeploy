# Shielded VMs
The data and state of a shielded VM are protected against inspection, theft and tampering from malware and datacenter administrators both at rest as well as in flight. 


Encryption and data at-rest/in-flight protection

    Virtual TPM (vTPM) enables the use of disk encryption within a VM (e.g. BitLocker).
    Both Live Migration and VM-state are encrypted.

Admin-lockout

    Host administrators cannot access guest VM secrets (e.g. can’t see disks, video, etc.)
    Allows you to Block unauthorized applications, malware, and debuggers from running.
    Host administrators cannot run arbitrary kernel-mode code on host.

Note: User mode code can be blocked with an appropriately configured CI policy, and we don't want to block all kernel-mode code, just foreign code that isn’t authorized to run there.

Attestation of health

    VM-workloads can only run on “healthy” hosts
![image](https://user-images.githubusercontent.com/71546848/220190199-ea577d5e-774d-4306-a76f-685ec01a65e8.png)
 

Virtual Machine Types

![image](https://user-images.githubusercontent.com/71546848/220190132-b813879c-b3ed-4b9b-8542-94c530aea6f8.png)



