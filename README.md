# KVM-Intallation-Guide-On-Debian-12

Here’s a GitHub README file for installing and configuring KVM on Debian 12:

---

# KVM Installation and Configuration on Debian 12

This guide provides a detailed, step-by-step process to install and configure KVM (Kernel-based Virtual Machine) on Debian 12.

## Prerequisites
- **BIOS/UEFI with Virtualization Enabled**: Ensure that virtualization (Intel VT-x or AMD-V) is enabled in your BIOS/UEFI settings.

## Installation Steps

### Step 0 : Optional (if you are using HyperV)

Steps to Enable Nested Virtualization
   1. **Shut down the VM**: Make sure the VM you want to enable nested virtualization on is powered off.
   2. **Run the PowerShell command**: On the Hyper-V host, run the following command to expose virtualization extensions for the VM:
 ```
Set-VMProcessor -VMName "YourVMName" -ExposeVirtualizationExtensions $true
```
Replace `YourVMName` with the name of your VM.

3. **Start the VM**: Power on the VM.

### Step 1: Verify Virtualization Support

1. **Enable Virtualization in BIOS**:
   - Restart your computer, enter BIOS/UEFI settings, and enable virtualization.
   - nested virtualization for hyperv
Nested virtualization is a feature that allows you to run Hyper-V inside a Hyper-V virtual machine (VM). This can be useful for testing configurations, running emulators, or creating isolated environments1. Here's how you can enable nested virtualization for Hyper-V:




2. **Check Virtualization in Debian**:
   ```bash
   lscpu | grep Virtualization
   ```
   - This should output information indicating virtualization support (e.g., `VT-x` for Intel or `AMD-V`).

### Step 2: Update System Packages

1. **Switch to Root User**:
   ```bash
   su -
   ```

2. **Update Package Lists**:
   ```bash
   apt update && apt upgrade -y
   ```

3. **Reboot if Required**:
   - If updates were significant, reboot your system.

### Step 3: Install KVM and Required Tools

Run the following command to install KVM, QEMU, libvirt, and Virt-Manager:
```bash
apt install qemu-kvm libvirt-daemon-system libvirt-clients virtinst virt-manager -y
```

### Step 4: Start and Enable the Libvirt Daemon

1. **Enable and Start Libvirt**:
   ```bash
   systemctl enable --now libvirtd
   ```

2. **Check Libvirt Status**:
   ```bash
   systemctl status libvirtd
   ```

### Step 5: Verify KVM Installation

1. **Check KVM Modules**:
   ```bash
   lsmod | grep kvm
   ```
   - You should see `kvm` and either `kvm_intel` or `kvm_amd`.

2. **Validate with `virsh`**:
   ```bash
   virsh list --all
   ```
   - This will list any active VMs (empty if none are created).

### Step 6: Validate System for KVM Compatibility

1. **Check KVM Modules Again**:
   ```bash
   lsmod | grep kvm
   ```

2. **Run `virt-host-validate`**:
   ```bash
   virt-host-validate
   ```
   - This checks for warnings or errors related to KVM support.

### Step 7: Optimize GRUB for IOMMU (Optional)

1. **Edit GRUB Configuration**:
   ```bash
   nano /etc/default/grub
   ```

2. **Update GRUB Parameters**:
   - For Intel, add `intel_iommu=on`:
     ```plaintext
     GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on"
     ```
   - Save and exit.

3. **Apply GRUB Changes**:
   ```bash
   update-grub
   ```

### Step 8: Create a Virtual Machine

#### Using Virt-Manager (GUI)

1. **Launch Virt-Manager**:
   ```bash
   virt-manager
   ```
2. **Create a New VM**:
   - Use the GUI to create a VM by following the setup prompts.

#### Using Command Line (Optional)

Use `virt-install` to create a VM:
```bash
virt-install \
  --name=myvm \
  --ram=2048 \
  --disk path=/var/lib/libvirt/images/myvm.img,size=20 \
  --vcpus=2 \
  --os-type linux \
  --os-variant debian11 \
  --network network=default \
  --graphics none \
  --location 'http://ftp.debian.org/debian/dists/bookworm/main/installer-amd64/' \
  --extra-args 'console=ttyS0,115200n8 serial'
```

### Step 9: Verify VM Functionality

1. **List Virtual Machines**:
   ```bash
   virsh list --all
   ```

2. **Monitor VM**:
   - Use Virt-Manager or `virsh` commands to manage and monitor your virtual machines.

---

This guide should provide everything needed to get KVM up and running on Debian 12.
