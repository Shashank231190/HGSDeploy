# Attestation modes in the Guarded Fabric solution
![image](https://user-images.githubusercontent.com/71546848/220191567-78ab163c-00ae-4fcb-90ff-a61840b8f7d6.png)

The Attestation Service does two things:
1. The Identity Attestation of the host 
The Service makes sure if this is the right host to trust. If the Service is using TPM, then it knows the ekPub it is requesting. If the Service is AD based, then the host needs to be part of a trusted AD Host Group. That is the HOST identity validation.
2. Measurements
The Attestation Service validates how the host is booted, whether the host has the right configuration, and if the configuration is trusted. Once both validations pass, then the Attestation Certificate is signed by the attestation signing key.

![image](https://user-images.githubusercontent.com/71546848/220191585-d95aedd1-5a0b-4ea7-8e86-7819bf7be7ac.png)

1. Admin Trusted attestation
2. TPM-trusted attestation (hardware-based)
3. Host key attestation (based on asymmetric key pairs)

![image](https://user-images.githubusercontent.com/71546848/220190750-ff95b1c2-7ed8-4787-8bd5-84191b6e893c.png)

