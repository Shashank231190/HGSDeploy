# Host Guardian Service (HGS)

An external authority in a guarded fabric that verifies the health of guarded hosts, and controls the release of keys required to start or live migrate Shielded VMs.

    A way to verify a host is in a healthy state
    A process to securely release keys to healthy hosts

HGS runs on a separate physical machine, typically in three instances. It runs two services: Attestation and Key Protection Service. The Hyper-V host contacts these services to get Attested, and to ask for the transport key to become unlocked.

Each HGS instance is a multi-instance web app, therefore you can have multiple instances—up to 64 in a single cluster.

![image](https://user-images.githubusercontent.com/71546848/220195667-80b590b9-a449-4ac1-b11a-a57b290329f4.png)

Guarded Hosts Verification

    The service verifies that only trusted fabric hosts that are pre-registered and identified by TPM hardware ID, will be authorized to run Shielded VMs in the fabric Or in the case of Admin-Attestation, that they are authorized hosts.
    
Remote Host Attestation

    The service performs attestation for the Guarded hosts in the fabric, to make sure that these hosts booted with the appropriate binaries, and have the right Hyper-V Code Integrity (HVCI) policy applied, It then provides the attested host with an attestation certificate to be used while requesting the Key Protector (KP) for a Shielded VM that needs to be launched

Key Protection

    A Shielded VM (when using BitLocker) can only be run by a host that is able to decrypt the vTPM of that VM (where the BitLocker key resides)
    A Guarded host should present its attestation certificate to request the decryptable Key Protector (KP) from the Key Protection Service (KPS), so that it can decrypt the vTPM.
