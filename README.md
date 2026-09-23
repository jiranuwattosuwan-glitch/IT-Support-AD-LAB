# IT-Support-AD-LAB
graph TD
    HOST[VMware Host 26H1u1] --- NET[VMnet NAT / Custom Network<br>Subnet: 192.168.10.0/24<br>Gateway: 192.168.10.1]
    NET --- DC01[LUNA-DC01<br>Windows Server 2025 Desktop Experience<br>IP: 192.168.10.10 Static<br>Status: Fresh OS Install]
