# Experiment  Create-a-datacenter-with-two-hosts-and-run-two-cloudlets-on-it.
Create a datacenter with two hosts and run two cloudlets on it. The cloudlets run in VMswith Different MIPS requirements. The cloudlets will take different time to complete the execution Depending on the requested VM performance.
 
## Overview
This project demonstrates a CloudSim 3.0.3 simulation experiment where two cloudlets are executed on virtual machines with different MIPS values.

Since the virtual machines have different computational capacities, the cloudlets complete execution at different times depending on VM performance.

This experiment is commonly used in cloud computing laboratory exercises, academic coursework, and performance analysis studies.

---

## Aim
To create a datacenter with two hosts and execute two cloudlets on virtual machines having different MIPS requirements such that cloudlets take different execution times.

---

## Objectives
- To simulate a cloud datacenter using CloudSim
- To create multiple hosts within a datacenter
- To configure virtual machines with different MIPS values
- To execute cloudlets with identical workloads
- To analyze execution time differences

---

## Tools and Technologies

| Tool | Version |
|------|---------|
| Java JDK | 1.8 |
| CloudSim | 3.0.3 |
| IDE | IntelliJ IDEA / Eclipse |
| Operating System | Windows / Linux |

---

## System Configuration

### Datacenter Configuration

| Parameter | Value |
|------------|-------|
| Datacenters | 1 |
| Hosts | 2 |
| RAM | 4096 MB |
| Storage | 1,000,000 MB |
| Bandwidth | 10,000 Mbps |
| VM Scheduler | Time Shared |

---

### Virtual Machine Configuration

| Parameter | VM1 | VM2 |
|------------|------|------|
| MIPS | 500 | 2000 |
| RAM | 512 MB | 512 MB |
| Bandwidth | 1000 Mbps | 1000 Mbps |
| VMM | Xen | Xen |
| Scheduler | Time Shared | Time Shared |

---

### Cloudlet Configuration

| Parameter | Value |
|------------|--------|
| Cloudlets | 2 |
| Length | 40,000 MI |
| File Size | 300 MB |
| Output Size | 300 MB |
| Processing Elements | 1 |

---

## Program File





package org.cloudbus.cloudsim.examples;

import java.text.DecimalFormat;
import java.util.ArrayList;
import java.util.Calendar;
import java.util.LinkedList;
import java.util.List;

import org.cloudbus.cloudsim.Cloudlet;
import org.cloudbus.cloudsim.CloudletSchedulerTimeShared;
import org.cloudbus.cloudsim.Datacenter;
import org.cloudbus.cloudsim.DatacenterBroker;
import org.cloudbus.cloudsim.DatacenterCharacteristics;
import org.cloudbus.cloudsim.Host;
import org.cloudbus.cloudsim.Log;
import org.cloudbus.cloudsim.Pe;
import org.cloudbus.cloudsim.Storage;
import org.cloudbus.cloudsim.UtilizationModel;
import org.cloudbus.cloudsim.UtilizationModelFull;
import org.cloudbus.cloudsim.Vm;
import org.cloudbus.cloudsim.VmAllocationPolicySimple;
import org.cloudbus.cloudsim.VmSchedulerSpaceShared;
import org.cloudbus.cloudsim.core.CloudSim;
import org.cloudbus.cloudsim.provisioners.BwProvisionerSimple;
import org.cloudbus.cloudsim.provisioners.PeProvisionerSimple;
import org.cloudbus.cloudsim.provisioners.RamProvisionerSimple;

public class CloudSimExample4 {

    private static List<Vm> vmlist;
    private static List<Cloudlet> cloudletList;

    public static void main(String[] args) {

        Log.printLine("Starting CloudSimExample4...");

        try {
            int numUser = 1;
            Calendar calendar = Calendar.getInstance();
            boolean traceFlag = false;

            CloudSim.init(numUser, calendar, traceFlag);

            // -------- CREATE ONE DATACENTER WITH TWO HOSTS --------
            Datacenter datacenter0 = createDatacenter("Datacenter_0");

            // -------- CREATE BROKER --------
            DatacenterBroker broker = createBroker();
            int brokerId = broker.getId();

            // -------- CREATE TWO VMs --------
            vmlist = new ArrayList<Vm>();

            int mips = 5000;
            long size = 10000;
            int ram = 512;
            long bw = 1000;
            int pesNumber = 1;
            String vmm = "Xen";

            Vm vm1 = new Vm(0, brokerId, mips, pesNumber, ram, bw, size, vmm,
                    new CloudletSchedulerTimeShared());

            Vm vm2 = new Vm(1, brokerId, mips, pesNumber, ram, bw, size, vmm,
                    new CloudletSchedulerTimeShared());

            vmlist.add(vm1);
            vmlist.add(vm2);

            broker.submitVmList(vmlist);

            // -------- CREATE TWO CLOUDLETS --------
            cloudletList = new ArrayList<Cloudlet>();

            long length = 40000;
            long fileSize = 300;
            long outputSize = 300;
            UtilizationModel utilizationModel = new UtilizationModelFull();

            Cloudlet cloudlet1 = new Cloudlet(0, length, pesNumber,
                    fileSize, outputSize,
                    utilizationModel, utilizationModel, utilizationModel);

            Cloudlet cloudlet2 = new Cloudlet(1, length, pesNumber,
                    fileSize, outputSize,
                    utilizationModel, utilizationModel, utilizationModel);

            cloudlet1.setUserId(brokerId);
            cloudlet2.setUserId(brokerId);

            cloudletList.add(cloudlet1);
            cloudletList.add(cloudlet2);

            broker.submitCloudletList(cloudletList);

            // -------- BIND CLOUDLETS TO VMs --------
            broker.bindCloudletToVm(0, 0);
            broker.bindCloudletToVm(1, 1);

            // -------- START SIMULATION --------
            CloudSim.startSimulation();

            List<Cloudlet> newList = broker.getCloudletReceivedList();

            CloudSim.stopSimulation();

            printCloudletList(newList);

            Log.printLine("CloudSimExample4 finished!");
        }
        catch (Exception e) {
            e.printStackTrace();
            Log.printLine("Simulation terminated due to error");
        }
    }

    // ================= DATACENTER WITH TWO HOSTS =================
    private static Datacenter createDatacenter(String name) {

        List<Host> hostList = new ArrayList<Host>();

        int mips = 5000;
        int ram = 2048;
        long storage = 1000000;
        int bw = 10000;

        // -------- HOST 1 --------
        List<Pe> peList1 = new ArrayList<Pe>();
        peList1.add(new Pe(0, new PeProvisionerSimple(mips)));

        Host host1 = new Host(
                0,
                new RamProvisionerSimple(ram),
                new BwProvisionerSimple(bw),
                storage,
                peList1,
                new VmSchedulerSpaceShared(peList1)
        );

        // -------- HOST 2 --------
        List<Pe> peList2 = new ArrayList<Pe>();
        peList2.add(new Pe(0, new PeProvisionerSimple(mips)));

        Host host2 = new Host(
                1,
                new RamProvisionerSimple(ram),
                new BwProvisionerSimple(bw),
                storage,
                peList2,
                new VmSchedulerSpaceShared(peList2)
        );

        hostList.add(host1);
        hostList.add(host2);

        String arch = "x86";
        String os = "Linux";
        String vmm = "Xen";
        double timeZone = 10.0;
        double cost = 3.0;
        double costPerMem = 0.05;
        double costPerStorage = 0.001;
        double costPerBw = 0.0;

        LinkedList<Storage> storageList = new LinkedList<Storage>();

        DatacenterCharacteristics characteristics =
                new DatacenterCharacteristics(
                        arch, os, vmm, hostList,
                        timeZone, cost,
                        costPerMem, costPerStorage, costPerBw
                );

        Datacenter datacenter = null;
        try {
            datacenter = new Datacenter(
                    name,
                    characteristics,
                    new VmAllocationPolicySimple(hostList),
                    storageList,
                    0
            );
        } catch (Exception e) {
            e.printStackTrace();
        }

        return datacenter;
    }

    private static DatacenterBroker createBroker() {

        DatacenterBroker broker = null;
        try {
            broker = new DatacenterBroker("Broker");
        } catch (Exception e) {
            e.printStackTrace();
        }
        return broker;
    }

    private static void printCloudletList(List<Cloudlet> list) {

        DecimalFormat dft = new DecimalFormat("###.##");
        Log.printLine("\n========== OUTPUT ==========");
        Log.printLine("Cloudlet ID    STATUS    Datacenter ID    VM ID    Time    Start    Finish");

        for (Cloudlet cloudlet : list) {
            if (cloudlet.getCloudletStatus() == Cloudlet.SUCCESS) {
                Log.printLine(
                        cloudlet.getCloudletId() + "           SUCCESS          " +
                        cloudlet.getResourceId() + "          " +
                        cloudlet.getVmId() + "      " +
                        dft.format(cloudlet.getActualCPUTime()) + "     " +
                        dft.format(cloudlet.getExecStartTime()) + "     " +
                        dft.format(cloudlet.getFinishTime())
                );
            }
        }
    }
}

---

## Algorithm

1. Initialize the CloudSim library.
2. Create a datacenter with two hosts.
3. Configure host resources.
4. Create a datacenter broker.
5. Create two virtual machines with different MIPS values.
6. Submit the VM list to the broker.
7. Create two cloudlets with identical workloads.
8. Assign cloudlets to VMs.
9. Start the CloudSim simulation.
10. Collect execution results and stop the simulation.

---

## Execution Time Formula

Execution Time = Cloudlet Length / VM MIPS

---

## Sample Output

<img width="697" height="78" alt="image" src="https://github.com/user-attachments/assets/b528ab36-b880-4627-afe4-e2f8f1d7549b" />

---

## Result

The experiment successfully demonstrates that cloudlets executed on higher MIPS virtual machines complete faster than those executed on lower MIPS VMs, even when the workload is identical.

---

## Conclusion

This simulation validates that CloudSim scheduling accurately reflects performance differences in heterogeneous cloud environments. Higher MIPS virtual machines reduce execution time for identical workloads.




