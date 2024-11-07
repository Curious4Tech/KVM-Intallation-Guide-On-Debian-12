# KVM-Intallation-Guide-On-Debian-12

Here’s a GitHub README file for installing and configuring KVM on Debian 12:

---

# KVM Installation and Configuration on Debian 12

This guide provides a detailed, step-by-step process to install and configure KVM (Kernel-based Virtual Machine) on Debian 12.

## Prerequisites
- **BIOS/UEFI with Virtualization Enabled**: Ensure that virtualization (Intel VT-x or AMD-V) is enabled in your BIOS/UEFI settings.

## Installation Steps

### Step 1: Verify Virtualization Support

1. **Enable Virtualization in BIOS**:
   - Restart your computer, enter BIOS/UEFI settings, and enable virtualization.
   - Steps to Enable Nested Virtualization (Optional, only for those who are using Hyper-V)
     
        1. **Shut down the VM**: Make sure the VM you want to enable nested virtualization on is powered off.
        2. **Run the PowerShell command**: On the Hyper-V host, run the following command to expose virtualization extensions for the VM:
 
      ```
       Set-VMProcessor -VMName "YourVMName" -ExposeVirtualizationExtensions $true
      ```

     Replace `YourVMName` with the name of your VM.
     
      3. **Start the VM**: Power on the VM.

2. **Check Virtualization in Debian**:
   ```bash
   egrep -c '(vmx|svm)' /proc/cpuinfo
   ```

   ![image](https://github.com/user-attachments/assets/105f76f7-cc36-4905-82fa-c94d8bb1c440)

   - If the output is `1` or more, your CPU supports hardware virtualization..

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
   
![image](https://github.com/user-attachments/assets/ca077942-d00f-4590-8908-4ab7b156c605)


### Step 5: Verify KVM Installation

1. **Check KVM Modules**:
   ```bash
   lsmod | grep kvm
   ```
   - You should see `kvm` and either `kvm_intel` or `kvm_amd`.


   ![image](https://github.com/user-attachments/assets/0a676daf-c614-4d02-91bf-b7bb122fca34)


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

   ![image](https://github.com/user-attachments/assets/c3738280-a689-40f7-84ec-8962ddb6b950)


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

### Step 8: Add Your User to the libvirt and kvm Groups

This step allows your user to manage VMs without root privileges.

Add your user to the libvirt and kvm groups:

```
    sudo usermod -aG libvirt,kvm $USER
```
 Log out and log back in for the group changes to take effect.

### Step 9: (Optional) Configure Networking for Virtual Machines

If you want your VMs to have network access, configure network bridging.

Check if the default network is active:

```
sudo virsh net-list --all
```
If default shows as inactive, start it:

```
sudo virsh net-start default
```

Ensure it starts on boot:

```
 sudo virsh net-autostart default
```

### Step 10: Launch Virt-Manager

To create and manage VMs using a graphical interface, launch Virt-Manager:

virt-manager

    In Virt-Manager, you can create new virtual machines, configure resources, and install operating systems.


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
