## GPU Passthrough Tutorial

In this post, I will be giving detailed instructions on how to run a KVM setup with GPU-passthrough (also referred to as "VGA passthrough" or "vfio" for the drivers). We will be using a Linux host (Pop!\_OS 19.10) and a Windows 10 guest VM.

### Considerations

The main reason I wanted to get this setup working was because I found myself tired of using a dual-boot setup. I wanted to launch a Windows VM specifically for gaming while still be able to use my Linux host for development work. 

At this point, you might be wondering... Why not just game on Linux? This is definitely an option for many people, but not one that suited my particular needs. Gaming on Linux requires the use of tools like [Wine](https://en.wikipedia.org/wiki/Wine_(software)) which act as a compatabilty layer for translating Windows system calls to POSIX-compliant system calls. A GPU-passthrough setup however uses [KVM](https://en.wikipedia.org/wiki/Kernel-based_Virtual_Machine) as a hypervisor to launch VMs. Performance wise, there are pros and cons to each approach. Check out [this thread](https://news.ycombinator.com/item?id=18328323) from Hacker News.

Many of the GPU-passthrough tutorials actually discuss completely different setups. We need to distinguish between the two types:

- Single GPU-Passthrough
    - This setup invovles *only* a single GPU that is passed from the host OS to the guest VM.
    - Most of the time, these type of setups occur when you have only a dGPU with no iGPU.
    - The host has to suspend it's display whenever the VM is active.
- Multiple GPU-Passthrough
    - This setup involves multiple GPUs, where a non-primary GPU is passed from the host to the guest VM while the primary GPU (usually an iGPU or 2nd GPU) is used for the host display
    - This setup requires at minimum 2 GPUs: one for the host and one for the VM.

For this tutorial, I will be sticking to a multi GPU-Passthrough setup. Specifically, I will be passing through an NVIDIA GPU to my guest VM while using an AMD GPU for my host.

### Hardware Requirements

### My Hardware Setup
- CPU:
    - Intel i7-8700k
- Motherboard:
    - ROG Strix Z370-E Motherboard
- GPUs:
    - NVIDIA RTX 2080 Ti
    - AMD RX590
    - Intel UHD 630 (iGPU)
- Memory:
    - Corsair Vengeance LPX DDR4 3200 MHz 32GB (2x16)
- Disk:
    - Samsung 970 EVO Plus SSD 500GB - M.2 NVME (host)
    - Samsung 970 EVO Plus SSD 1TB - M.2 NVME (guest)

### Credits + Useful Resources

- [Heiko Sieger - Running Windows 10 on Linux using KVM with VGA Passthrough](https://heiko-sieger.info/running-windows-10-on-linux-using-kvm-with-vga-passthrough)
    - An excellent resource. Written for setups with 2 GPUs: 1 iGPU + 1 dGPU or 2 dGPUs. 
    - Unlike the tutorial here, Heiko's binds the dGPU at boot time rather than dynamically before VM starts. This is fine for most setups, but not for those who want to use their dGPU on a host whenever the VM shutdown. *TODO: link to bind/unbind vfio section.*
    - The same goes for hugepages. Heiko's tutorial allocates hugepages at boot-time whereas this tutorial does it dynamically. Again this is fine for most setups, but those who want to free up RAM space whenever their VM is shutdown benefit more from dynamic allocation. *TODO: link to hugepage section.*
- [The Passthrough Post](https://passthroughpo.st/) - A blog dedicated to the latest PCI passthrough/VFIO related news, guides, benchmarks and tools
    - [VFIO PC Builds](https://passthroughpo.st/vfio-increments/) - a list of parts for VFIO-focused PC builds at different price ranges