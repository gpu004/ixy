# Agent notes

`README.md` covers what ixy is, how to build it (`cmake . && make`, after `sudo ./setup-hugetlbfs.sh`) and how to run `ixy-fwd` and `ixy-pktgen`.

## Layout

- `src/driver/ixgbe.c`, `ixgbe.h`: Intel 82599/X5xx driver. `ixgbe_type.h` holds register definitions copied from DPDK and is not counted toward the "1000 lines".
- `src/driver/virtio.c`: VirtIO (legacy) driver for VMs.
- `src/driver/device.{c,h}`: driver-independent device API (`ixy_init`, `ixy_rx_batch`, `ixy_tx_batch`).
- `src/memory.c`: hugepage DMA memory and mempools (single-threaded, not thread-safe).
- `src/pci.c`, `src/libixy-vfio.c`, `src/interrupts.c`: PCI/sysfs access, optional VFIO/IOMMU, interrupts.
- `src/app/ixy-fwd.c`: forwards **in both directions** on one core (`forward(dev1→dev2)` and `forward(dev2→dev1)` in the same loop). Per-core throughput is the sum of both ports' RX Mpps.
- `src/app/ixy-pktgen.c`, `ixy-pcap.c`: packet generator and pcap capture.

## Benchmarks and performance claims

- Real line-rate numbers need a physical ixgbe NIC bound to ixy (no kernel driver). Cloud/Devin VMs have no NIC passthrough, so numbers from there are not comparable to the paper. Say so explicitly when reporting.
- Primary sources for cycle-per-packet claims:
  - ixy paper (Emmerich et al., ANCS 2019): https://www.net.in.tum.de/fileadmin/bibtex/publications/papers/ixy-writing-user-space-network-drivers.pdf. It has ixy vs DPDK cycles/packet and the per-stage breakdown.
  - "Comparison of Frameworks for High-Performance Packet IO" (ICN 2015): https://www.net.in.tum.de/fileadmin/bibtex/publications/papers/ICN2015.pdf. Table II gives the Linux per-packet cost by stage (RX, TX, memory, `sk_buff`, processing).
- When checking a quoted number, cite the table and say which quantity it measures: one-way or bidirectional, cycles per packet or per batch, and the driver and batch size.
