---
title: Virtual TPMs in Azure confidential VMs
description: Learn about how we use virtual Trusted Platform Modules in our confidential VMs.
author: simranparkhe
ms.author: simranparkhe
ms.service: azure-virtual-machines
ms.subservice: azure-confidential-computing
ms.topic: concept-article
ms.date: 07/20/2023
ms.custom: template-concept
# Customer intent: "As a cloud security administrator, I want to understand how virtual Trusted Platform Modules work in Azure confidential VMs, so that I can ensure robust security and integrity for the applications running in my organization’s cloud infrastructure."
---

# Virtual TPMs in Azure confidential VMs

A [Trust Platform Module (TPM)](/windows/security/information-protection/tpm/trusted-platform-module-overview) is designed to provide hardware based security functions. These functions include secret storage for cryptographic keys, storage for measurements of the boot process and an external hardware root-of-trust. 

Azure confidential VMs each have their own dedicated virtual TPM (vTPM). The vTPM is a virtualized version of a hardware TPM, and complies with the [TPM2.0 spec](/windows/security/information-protection/tpm/tpm-recommendations#why-tpm-20). In a confidential VM, the vTPM runs inside the VM in a hardware-based protected memory region. With this architecture, each confidential VM has its own unique vTPM instance that is isolated and encrypted by AMD SEV-SNP. Thus, an Azure confidential VM's vTPM instance is isolated from the hosting environment and all other VMs on the system.

:::image type="content" source="media/vtpm-docs/vtpm-in-confidential-vm-diagram.png" alt-text="Diagram of a confidential VM showing where the vTPM runs, how it's measured and how it's isolated.":::

For more information on the technology, see our blog on [confidential VMs](https://techcommunity.microsoft.com/t5/windows-os-platform-blog/confidential-vms-on-azure/ba-p/3836282).

Since the vTPM runs within the Confidential VM, it's measured by the AMD SEV-SNP hardware. Customers can retrieve the hardware report generated by the Platform Security Processor (PSP) to attest the identity and integrity of the vTPM that guarantees that the TPM is authentic.

The hardware report can be used to verify that the Confidential VM is running as an isolated, secure machine with an isolated, integrity protected and measured vTPM. The vTPM can in-turn be used to measure and securely the boot the OS components in the confidential VM. By using usual TPM based primitives such as Measured Boot and [Secured Boot](/windows-hardware/design/device-experiences/oem-secure-boot), you can ensure and prove that your confidential VM is launched as intended.

TPMs have platform configuration registers (PCRs) that can be used to cryptographically measure the software state to ensure that nothing has been tampered with or misused. The [PCR values](/windows/security/hardware-security/tpm/switch-pcr-banks-on-tpm-2-0-devices) are one-way hashes to ensure that the measurements can't be removed or altered. PCRs are used to store the measurements of various boot artifacts to help with the [measured boot process](/azure/security/fundamentals/measured-boot-host-attestation). PCRs can also be used to measure applications, disk integrity measurements and other components. Additionally, PCRs can be used to enforce security policies such as application and code integrity (CI) policies to ensure the system remains compliant with the desired policies.

To utilize vTPMs in confidential VMs further, see [How to leverage vTPMs in confidential VMs](how-to-leverage-virtual-tpms-in-azure-confidential-vms.md).
