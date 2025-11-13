# Module 7: Assignment: Maintaining Secure Systems with SBOMs and Patching

### **Exercise Overview**
In Module 7, you explored how assurance, verification, and transparency strengthen system design and maintenance. This lab assignment extends those concepts into the realm of secure maintenance, demonstrating how visibility through SBOMs enables continuous verification of the operating environment itself.

Rather than scanning application code, you will analyze the Ubuntu Linux system that powers your GitHub Codespace - enumerating installed packages, identifying vulnerabilities, applying patches, and validating the system’s security posture using a second SBOM and vulnerability scan. 

In their Security Engineering principles, <b><a href="https://doi.org/10.1109/PROC.1975.9939" target="_blank">(Saltzer & Schroeder, 1975)</a></b> emphasized that every access — and by extension, every component — must be continuously verified to maintain trust. By applying this principle to your Codespaces environment, the lab illustrates how assurance extends beyond applications to the underlying platform itself. Through system-level SBOM generation, vulnerability assessment, and patch verification, you will connect visibility (SBOM) with action (patching) — strengthening the assurance needed for secure operations.

### **Learning Objectives**
After completing this lab, you will be able to:

1. Generate an SBOM of the system packages installed in your Codespace.  
2. Identify vulnerabilities affecting the base Ubuntu environment.  
3. Apply and verify system updates using `apt`.  
4. Explain how patching operational environments supports *complete mediation* and *assurance* in the secure design lifecycle.

### **Terminology**
**Packages** refer to the individual software components installed on your Ubuntu-based Codespace. A package is essentially a bundled unit of software, containing:
* The program or library files,
* Metadata (version number, description, dependencies), and
* Instructions for installing, configuring, and removing it.

 These are managed by Ubuntu’s package manager (`apt`) and include:
* System utilities (e.g., `bash`, `coreutils`, `curl`),
* Programming languages and runtimes (e.g., `python3`, `gcc`),
* Libraries used by other programs (e.g., `libssl`, `libc6`), and
* Tools and services required by the operating system.
  
In this assignment, when you run commands like:

```bash
dpkg -l
apt list --installed
syft scan:apt
```

...you are examining all the packages your Codespace depends on to function.
These packages form the software supply chain of your operating environment — and generating SBOMs, scanning for CVEs, and applying updates allow you to evaluate and improve its security posture.

**apt**, short for **Advanced Package Tool**, is the package management system used by Ubuntu and other Debian-based Linux distributions. It is responsible for:
* Installing new software,
* Updating existing software,
* Removing software,
* Downloading and applying security patches, and
* Managing dependencies between packages.

In this lab, you use `apt` to:
* Refresh the system’s package index (`sudo apt update`),
* View which packages can be upgraded (`sudo apt list --upgradable`), and
* Apply updates and security patches (`sudo apt upgrade -y`).

These actions are essential for:
* Maintaining system integrity,
* Ensuring outdated or vulnerable software is replaced, and
* Supporting continuous verification and assurance (key Module 7 concepts).

### **Environment Setup**
All commands can be executed directly inside your GitHub Codespace using this repo.  

#### **Part 1 – Generate Baseline System Information**
Before generating SBOMs, let's first collect information about the current Ubuntu system in your Codespace:

1. Check the Linux and Ubuntu version:

   ```bash
   uname -a
   cat /etc/os-release
   ```
   
2. View currently installed packages and save the output in *before_patch.txt* inside the `deliverables` folder:
    
   ```bash
   dpkg -l > deliverables/before_patch.txt
   ```

| **Syntax Breakdown** | **Description**
|---------------|------------------------|
| **dpkg** | Package manager for Debian
| **-l** | Program option (*-l*) used to *list* installed packaages 
| **> deliverables/before_patch.txt** | Re-direct the output to a file named *before_patch.txt* in the `deliverables` folder

#### **Part 2 – Generate Baseline SBOM and Vulnerability Report**
1. Use Syft to create a system-level SBOM of APT-managed packages (**NOTE**: This scan will take approximately 3 minutes to complete):

```bash
syft scan dir:/ -o spdx-json > deliverables/system_sbom_before.json
```

| **Syntax Breakdown** | **Description**
|---------------|------------------------|
| **syft** | Program used to create an SBOM in an .spdx format
| **scan** | Program sub-command used to *scan* the Codespace’s Ubuntu filesystem for installed packages
| **dir:/** | The directory to be scanned; in this case, root filesystem (/)
| **-o spdx-json** | Output (*-o*) the results in an *SPDX JSON* format
| **> deliverables/system_sbom_before.json**| Standard shell redirection (*>*) that save the previous line's results in a file named *system_sbom_before.json* in the `deliverables` folder

2. Use Grype to scan the SBOM for known vulnerabilities:

```bash
grype sbom:deliverables/system_sbom_before.json -o table > deliverables/system_vulns_before.txt
```

| **Syntax Breakdown** | **Description**
|---------------|------------------------|
| **grype** | Program used to scan something for vulnerabilities
| **sbom:deliverables/system_sbom_before.json** | Program sub-command (*sbom*) telling Grype what to scan; in this case, the SBOM file generated in the previous step
| **-o table** | Output (*-o*) the results as a human-readable *table*
| **> deliverables/system_vulns_before.txt**| Standard shell redirection (*>*) that takes the previous command and saves it in a file names *system_vulns_before.txt* in the `deliverables` folder

3. Review and record:
* Total number of packages detected,
* Number of vulnerabilities by severity (Critical, High, Medium, Low),
* In your `reflection.md` file, which you will create in the `deliverables` folder, select one CVE, locate it in the <b><a href="https://nvd.nist.gov/vuln">NVD Database</a></b>, and summarize its cause or impact in a few sentences.

#### **Part 3 – Identify and Apply System Updates**
1. Update the package database:
   
   ```bash
   sudo apt update
   ```
   
2. List available updates

   ```bash
   sudo apt list --upgradable
   ```
   
3. If packages are listed, choose at least three (3) and record their current and target versions. Continue with the next step (4. Apply system updates). **NOTE**: If no packages are listed (only `“Listing… Done”`), record that your system is already fully up-to-date and note this in your reflection and jump to Part 4: Post-Update Analysis.
   
4. Apply system updates:

   ```bash
   sudo apt upgrade -y
   ```
5. Verify updates were applied successfully:

    ```bash
    sudo apt list --upgradable
    ```
(If no packages are listed, all updates were applied).

#### **Part 4 – Post-Update Analysis**
1. Take a second snapshot - save the updated package list as *after_patch.txt* inside your `/deliverables` folder:

   ```bash
   dpkg -l > deliverables/after_patch.txt
   ```
2. Regenerate the SBOM after updates (**NOTE**: This scan will take approximately 3 minutes to complete):

   ```bash
   syft scan dir:/ -o spdx-json > deliverables/system_sbom_after.json
   ```

3. Re-scan for vulnerabilities:

   ```bash
   grype sbom:deliverables/system_sbom_after.json -o table > deliverables/system_vulns_after.txt
   ```

4. Compare the before and after snapshots: Use the `diff` command to view changes between *before_patch.txt* and *after_patch.txt*:

   ```bash
   diff deliverables/before_patch.txt deliverables/after_patch.txt > deliverables/patch_diff.txt
   ```

5. Review the resulting file (*patch_diff.txt*) to identify which packages were updated or removed (**NOTE**: *patch_diff.txt* will be empty if no updates were available).

#### **Part 5 – Reflection**
Answer the following in 2–3 sentences each:

1. How does maintaining updated packages illustrate *complete mediation*?  
2. Why is generating a new SBOM after patching essential to assurance?  
3. What risks persist when organizations delay system updates?  
4. How does this exercise embody the *secure design lifecycle* principle?

### **Deliverables**
**GitHub Submission**:

All files should be saved in your `/deliverables` folder and committed/pushed to your forked GitHub repository:

```
deliverables/
│
├── before_patch.txt – Package list before updates
├── after_patch.txt – Package list after updates
├── patch_diff.txt – Comparison of before/after package lists
├── system_sbom_before.json and system_sbom_after.json – SBOMs generated by Syft
├── system_vulns_before.txt and system_vulns_after.txt – Vulnerability reports from Grype
└── reflection.md - You create this file; it's your one-page written reflection, answering any of the questions above
```

**Canvas Submission**:

1. Copy your forked repository URL: Go to your forked repo on GitHub (it should look something like):
   
     `https://github.com/<your-username>/eng298-fa25-mod7-sbom-lab2`
   
   - Copy the URL from your browser’s address bar.

3. Submit the URL in Canvas: Module 7: Assignment: Maintaining Secure Systems with SBOMs and Patching

### **Grading (20 Points Total)**

| **Category** | **Description** | **Points** |
|---------------|-----------------|------------|
| **Baseline Scan** | Complete SBOM and vulnerability report generated before patching | 5 |
| **Update Process** | Correct use of apt update / apt upgrade with verification and snapshots | 5 |
| **Post-Update Scan** | Clear comparison showing effects of updates on packages and vulnerabilities | 5 |
| **Reflection & Clarity** | Concise explanation connecting patching to security-engineering principless | 5 |

