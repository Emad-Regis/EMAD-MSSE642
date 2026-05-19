Course: MSSE 642 - Software Assurance

Instructor: Randall Granier

Group 2

Author: Emad Fattah

Version: 3

Table of Contents

1\. Overview

2\. Technology Stack

3\. Architectural Diagram

4\. Virtualization Environment

5\. Kali Linux Setup

6\. Nessus Installation

7\. Metasploitable 2 Setup

8\. Network Connectivity Test

9\. Problems & Solutions

**1\. Overview:**

This write-up documents the setup of a local Penetration Testing lab environment for MSSE 642. The goal was to create a minimal but functional pentest lab running on a local machine using virtualization. The lab consists of two virtual machines:

**Kali Linux:** The attacker machine, used for penetration testing.

**Metasploitable 2**: The intentionally vulnerable target machine used as the attack surface.

This environment mirrors industry-standard pen testing setups and will be used for upcoming assignments in Weeks 6 and 8.

## 2\. Technology Stack

| Component       | Details                               |
| --------------- | ------------------------------------- |
| Host Machine    | mOacOC                                |
| Host OS         | MacOS Sequoia Verstion 15.7.3         |
| Attacker VM     | Kali Linux 2026.1-installer-arm64.iso |
| Target VM       | Metasploitable 2 (x86_64)             |
| VM Network Mode | Isolated lab network                  |

## **Note on Hypervisor Choice**:UTM is used on Apple devices - macOS, iOS, and visionOS - because it's an open-source virtual machine and emulator built specifically for Apple's hardware and software ecosystem. It leverages Apple's built-in hypervisor and QEMU to run a wide range of operating systems, from modern Windows and Linux to older PowerPC, SPARC, and ARM versions

3\. Architectural Diagram

The Digram created using Nano Bannan 2 ([http://arlist.io](http://arlist.io/)). The diagram shows the Windows host running VirtualBox with two isolated VMs connected via a Host-Only virtual networ

4\. Virtualization Environment

UTM was downloaded and installed from the UTM website (<https://Mac.getutm.app>). It is free and open-source.

Steps taken:

1\. Downloaded the UTM Virtual machines for Mac from \`Mac.getutm.app\`.

2\. Ran the installer and completed the setup wizard with default options.

5\. Kali Linux Setup

Steps taken:

1\. Downloaded the Kali Linux VirtualBox image (\`.iso\`) from \`kali.org\`.

2\. In UTM, went to Start → Virtualize → Linux and selected the image \`.iso\` file.

3\. Completed the import wizard with default settings.

4\. Assigned the Host-Only network adapter to the VM under Settings → Network.

5\. Booted the VM and logged in with username a (\`efmse642\`) and new password.

6\. Updated the system:

6\. Nessus Installation:

Nessus Essentials (free tier) was installed on the Kali Linux VM for vulnerability scanning.

Steps taken:

1\. Registered for a free Nessus Essentials activation code at \[tenable.com/products/nessus/nessus-essentials\](<https://www.tenable.com/products/nessus/nessus-essentials>).

2\. Downloaded the Nessus \`.deb\` package for Debian/Kali (Nessus-10.12.0-ubuntu1804_aarch64.deb) from the Tenable downloads page.

3\. Installed the package:

\`\`\`bash

sudo dpkg -i Nessus-10.12.0-ubuntu1804_aarch64.deb

\`\`\`

4\. Started the Nessus service:

\`\`\`bash

sudo systemctl start nessusd

sudo systemctl enable nessusd

\`\`\`

5\. Navigated to \`<https://kali:8834\`> in the Kali browser to complete the setup wizard and enter the activation code.

6\. Waited for the initial plugin compilation to complete.

7\. Metasploitable 2 Setup

## Steps taken

1\. Downloaded Metasploitable 2 from (<https://sourceforge.net/projects/metasploitable/>).

3\. Opened UTM , created a new VM (\*New → Emulate → Other ) and attached the \`.vmdk\` as the existing virtual hard disk.

4\. Assigned the Host-Only network adapter under \*Settings → Network\*.

5\. Booted the VM; logged in with default credentials (\`msfadmin\` / \`msfadmin\`).

6\. Noted the IP address assigned by the Host-Only network:

\`\`\`bash

ifconfig

\`\`\`

8\. Network Connectivity Test:

To verify that the Kali Linux attacker machine can reach the Metasploitable 2 target, a \`ping\` test was performed from within the Kali Linux VM.

Steps taken:

1\. Identified the Metasploitable 2 IP from its terminal (\`ifconfig\`).

2\. From the Kali Linux terminal, ran:

\`\`\`bash

ping -c 5 &lt;metasploitable-ip&gt;

\`\`\`

(Replace \`&lt;metasploitable-ip&gt;\` with the actual IP, e.g., \`192.168.64.6\`)\*

## 9\. Problems & Solutions

## 1\. Selecting the Hypervisor (VirtualBox vs. UTM)

- The Challenge: Initial attempts to establish the Kali Linux virtual machine using Oracle VirtualBox on the M4 Pro MacBook failed due to compatibility roadblocks. VirtualBox does not natively or smoothly support virtualization on Apple Silicon ARM-based chips.
- The Solution: Following extensive research and technical reviews, I pivoted to UTM. UTM leverages Apple's native Hypervisor framework and QEMU. This offered a significantly smoother, faster, and more stable performance tailored to the Mac hardware architecture.

## 2\. Kali Linux Installation Optimization

- The Challenge: Deploying Kali Linux inside UTM presented various configuration methodologies, several of which led to installation glitches, hanging screens, or file system errors.
- The Solution: Through trial and error, I identified the optimal deployment workflow. By avoiding manual virtual disk partitioning during the Debian/Kali setup wizard and allowing the installer to use a default, unpartitioned single-drive layout, the operating system installed with zero technical glitches.

## 3\. Metasploitable 2 Configuration & Verification

- The Challenge: Because Metasploitable 2 is an older, legacy x86_64 Linux environment, running and managing it via emulation on an ARM64 Apple Silicon chip presented initial stability and control hurdles.
- The Solution: I successfully resolved these configuration anomalies by ensuring correct terminal communication with the underlying OS. To verify that the VM state, kernel integration, and hypervisor bindings were operating flawlessly, I tested the system power management. Executing a clean(poweroff)command without system hangs or kernel panics confirmed that the emulation environment was stable and correctly established.

4\. Nessus installation

- I have to find out which version is good for working Apple MacBook, I found out (Nessus-10.12.0-ubuntu1804_aarch64.deb) is functional rather than (Nessus-10.12.0-debian10_am64.deb) and that cost me a lot of time.

References:

\- Kali Linux Official Site: \[<https://www.kali.org\>](<https://www.kali.org>)

\- Oracle VirtualBox: \[<https://www.virtualbox.org\>](<https://www.virtualbox.org>)

\- Tenable Nessus Essentials: \[<https://www.tenable.com/products/nessus/nessus-essentials\>](<https://www.tenable.com/products/nessus/nessus-essentials>)

\- Metasploitable 2 Download: \[<https://sourceforge.net/projects/metasploitable/\>](<https://sourceforge.net/projects/metasploitable/>)