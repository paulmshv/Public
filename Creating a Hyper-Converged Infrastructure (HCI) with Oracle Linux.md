Creating a **Hyper-Converged Infrastructure (HCI)** with Oracle Linux involves combining compute, storage, and networking into a single, unified system. Oracle Linux is a robust platform that supports virtualization technologies like **KVM** and **Oracle VM**, and it can be integrated with distributed storage solutions like **Ceph** or **GlusterFS** to achieve HCI.

Here’s a step-by-step guide to creating an HCI virtualization environment with Oracle Linux:

---

### **1. Prerequisites**
- **Hardware**: 4 or more bare metal servers with identical hardware (CPU, RAM, storage).
- **Oracle Linux**: Install Oracle Linux 8 or 9 on all servers.
- **Network**: Ensure proper networking (1 Gbps or 10 Gbps) with VLANs for storage, management, and VM traffic.
- **Shared Storage**: Use distributed storage like Ceph or GlusterFS for HCI.

---

### **2. Install Oracle Linux**
1. Download Oracle Linux from the [Oracle Linux website](https://www.oracle.com/linux/).
2. Install Oracle Linux on all servers.
   - Choose the "Minimal Install" or "Server with GUI" option.
   - Ensure all servers have static IP addresses.
3. Update the system:
   ```bash
   sudo dnf update -y
   ```

---

### **3. Set Up Virtualization (KVM)**
1. Install KVM and virtualization tools:
   ```bash
   sudo dnf install -y qemu-kvm libvirt virt-install virt-manager bridge-utils
   ```
2. Start and enable the libvirtd service:
   ```bash
   sudo systemctl start libvirtd
   sudo systemctl enable libvirtd
   ```
3. Verify KVM installation:
   ```bash
   lsmod | grep kvm
   ```
4. Configure networking for VMs:
   - Create a bridge interface for VM networking.
   - Edit `/etc/sysconfig/network-scripts/ifcfg-br0`:
     ```
     DEVICE=br0
     TYPE=Bridge
     BOOTPROTO=static
     IPADDR=<server-ip>
     NETMASK=<netmask>
     GATEWAY=<gateway>
     DNS1=<dns-server>
     ONBOOT=yes
     ```
   - Restart the network service:
     ```bash
     sudo systemctl restart network
     ```

---

### **4. Set Up Distributed Storage (Ceph or GlusterFS)**
#### **Option 1: Ceph**
1. Install Ceph on all nodes:
   ```bash
   sudo dnf install -y ceph ceph-radosgw
   ```
2. Configure Ceph:
   - Create a Ceph cluster:
     ```bash
     ceph-deploy new <node1> <node2> <node3>
     ```
   - Install Ceph on all nodes:
     ```bash
     ceph-deploy install <node1> <node2> <node3>
     ```
   - Deploy the monitor and manager:
     ```bash
     ceph-deploy mon create-initial
     ceph-deploy mgr create <node1>
     ```
   - Add OSDs (Object Storage Daemons):
     ```bash
     ceph-deploy osd create --data /dev/sdb <node1>
     ceph-deploy osd create --data /dev/sdb <node2>
     ceph-deploy osd create --data /dev/sdb <node3>
     ```
3. Verify the Ceph cluster:
   ```bash
   ceph -s
   ```

#### **Option 2: GlusterFS**
1. Install GlusterFS on all nodes:
   ```bash
   sudo dnf install -y glusterfs-server
   ```
2. Start and enable GlusterFS:
   ```bash
   sudo systemctl start glusterd
   sudo systemctl enable glusterd
   ```
3. Create a trusted storage pool:
   ```bash
   sudo gluster peer probe <node2>
   sudo gluster peer probe <node3>
   ```
4. Create a distributed volume:
   ```bash
   sudo gluster volume create gv0 replica 3 <node1>:/data/brick1 <node2>:/data/brick2 <node3>:/data/brick3
   sudo gluster volume start gv0
   ```

---

### **5. Integrate Storage with Virtualization**
1. For **Ceph**:
   - Create an RBD (RADOS Block Device) pool for VM disks:
     ```bash
     ceph osd pool create vm-pool 128
     ```
   - Configure KVM to use Ceph RBD:
     - Install `rbd` tools:
       ```bash
       sudo dnf install -y ceph-common
       ```
     - Add the RBD pool to KVM:
       ```xml
       <disk type='network' device='disk'>
         <source protocol='rbd' name='vm-pool/vm-disk'>
           <host name='<ceph-monitor>' port='6789'/>
         </source>
         <target dev='vda' bus='virtio'/>
       </disk>
       ```

2. For **GlusterFS**:
   - Mount the GlusterFS volume on all nodes:
     ```bash
     sudo mount -t glusterfs <node1>:/gv0 /mnt/glusterfs
     ```
   - Use the mounted directory as storage for VM disks.

---

### **6. Set Up High Availability (HA)**
1. Use **Pacemaker** and **Corosync** for HA:
   ```bash
   sudo dnf install -y pacemaker pcs
   ```
2. Configure the cluster:
   ```bash
   sudo pcs cluster auth <node1> <node2> <node3> -u hacluster -p <password>
   sudo pcs cluster setup --name mycluster <node1> <node2> <node3>
   sudo pcs cluster start --all
   ```
3. Add resources (e.g., virtual IP, services) to the cluster.

---

### **7. Deploy Virtual Machines**
1. Use `virt-install` to create VMs:
   ```bash
   sudo virt-install \
   --name vm1 \
   --ram 2048 \
   --vcpus 2 \
   --disk path=/var/lib/libvirt/images/vm1.qcow2,size=20 \
   --os-type linux \
   --os-variant oraclelinux8 \
   --network bridge=br0 \
   --graphics none \
   --console pty,target_type=serial \
   --location /path/to/oracle-linux.iso \
   --extra-args 'console=ttyS0,115200n8 serial'
   ```
2. Manage VMs using `virt-manager` or `virsh`.

---

### **8. Monitor and Maintain**
- Use **Grafana** and **Prometheus** for monitoring.
- Regularly update Oracle Linux and all software.
- Perform backups of VMs and critical data.

---

By following these steps, you can create a robust HCI virtualization environment using Oracle Linux, KVM, and distributed storage like Ceph or GlusterFS. This setup will provide scalability, high availability, and efficient resource utilization for your production environment.
