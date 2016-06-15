---
layout: post
title: Google Compute Engine with gcloud-java
description: This post shows how to use gcloud-java, the Java idiomatic client for Google Cloud Platform services, to manage your Google Compute Engine resources.
keywords: gcloud-java, Google Compute Engine, GCE, Java, cloud, VM, virtual machine, disk, snapshot, image, code snippets
---

Google Cloud Compute allows users to use virtual machines (VM) running in Google's data centers and fiber network. Compute Engine's tools enable scaling from single instances to global, load-balanced cloud computing.  

[gcloud-java](https://github.com/GoogleCloudPlatform/gcloud-java) is the Java idiomatic client for
Google Cloud Platform services. `gcloud-java` aims at providing Java developers with a smooth experience
when interacting with Google Cloud services. For infos on how to add `gcloud-java` to your project and start
using it see the project's [README](https://github.com/GoogleCloudPlatform/gcloud-java/blob/master/README.md).

Recently, `gcloud-java` added alpha support for [Google Compute Engine](https://cloud.google.com/compute/).
In this post we will go through some of the main features that the client now supports.

## Create a Persistent Disk and a Snapshot

Google Compute Engine's VMs support persistent storage through disks. You can create persistent disks
and attach them your instances according to your needs. Compute Engine offers many public images that 
can be used to create boot disks for your VMs with Linux and Windows operating systems (a
complete list can be found [here](https://cloud.google.com/compute/docs/images#os-compute-support)).

Using `gcloud-java` you can easily create a persistent disk from an existing image, see the following code
for an example:

```java
Compute compute = ComputeOptions.defaultInstance().service();
ImageId imageId = ImageId.of("debian-cloud", "debian-8-jessie-v20160329");
DiskId diskId = DiskId.of("us-central1-a", "test-disk");
ImageDiskConfiguration diskConfiguration = ImageDiskConfiguration.of(imageId);
DiskInfo disk = DiskInfo.of(diskId, diskConfiguration);
operation = compute.create(disk);
// Wait for operation to complete
operation = operation.waitFor();
if (operation.errors() == null) {
  System.out.println("Disk " + diskId + " was successfully created");
} else {
  // inspect operation.errors()
  throw new RuntimeException("Disk creation failed");
}
```

Once a disk has been created, `gcloud-java` can also be used to create disk snapshots.
Snapshots can be used for periodic backups of your disks and can be created even while the disk is attached to a
running VM. Snapshots are differential saves of your disk state, which means that they can be created faster and at a lower cost when compared to disk images.

To create a snapshot for an existing disk in `gcloud-java` the following code can be used:

```java
DiskId diskId = DiskId.of("us-central1-a", "test-disk");
Disk disk = compute.getDisk(diskId, Compute.DiskOption.fields());
if (disk != null) {
  String snapshotName = "test-disk-snapshot";
  Operation operation = disk.createSnapshot(snapshotName);
  // Wait for operation to complete
  operation = operation.waitFor();
  if (operation.errors() == null) {
    // use snapshot
    Snapshot snapshot = compute.getSnapshot("test-disk-snapshot");
  }
}
```

## Create an External IP Address

External addresses can be assigned to a VM to allow communication with outside of Compute Engine:
a specific instance can be reached via its external IP address, as long as there is an existing
firewall rule to allow it. External addresses can be either ephemeral or static: ephemeral addresses
change everytime the instance is restarted or terminated; static addresses, instead, are tied to a project
until you explicitly release them. `gcloud-java` makes creating a static external address as simple as
typing the following code:

```java
RegionAddressId addressId = RegionAddressId.of("us-central1", "test-address");
Operation operation = compute.create(AddressInfo.of(addressId));
// Wait for operation to complete
operation = operation.waitFor();
if (operation.errors() == null) {
  System.out.println("Address " + addressId + " was successfully created");
} else {
  // inspect operation.errors()
  throw new RuntimeException("Address creation failed");
}
```

## Create a Compute Engine VM

Once a disk and a static external address have been created, we have everything we need to create and start
a fully operational Compute Engine VM. The following `gcloud-java` code creates and starts a VM
instance using the just-created disk and address:

```java
Address externalIp = compute.getAddress(addressId);
InstanceId instanceId = InstanceId.of("us-central1-a", "test-instance");
NetworkId networkId = NetworkId.of("default");
PersistentDiskConfiguration attachConfiguration =
    PersistentDiskConfiguration.builder(diskId).boot(true).build();
AttachedDisk attachedDisk = AttachedDisk.of("dev0", attachConfiguration);
NetworkInterface networkInterface = NetworkInterface.builder(networkId)
    .accessConfigurations(AccessConfig.of(externalIp.address()))
    .build();
MachineTypeId machineTypeId = MachineTypeId.of("us-central1-a", "n1-standard-1");
InstanceInfo instance =
    InstanceInfo.of(instanceId, machineTypeId, attachedDisk, networkInterface);
operation = compute.create(instance);
// Wait for operation to complete
operation = operation.waitFor();
if (operation.errors() == null) {
  System.out.println("Instance " + instanceId + " was successfully created");
} else {
  // inspect operation.errors()
  throw new RuntimeException("Instance creation failed");
}
```

You can also create a VM instance using an ephemeral IP address. An ephemeral address will
be automatically assigned to the instance when using the following code:

```java
Address externalIp = compute.getAddress(addressId);
InstanceId instanceId = InstanceId.of("us-central1-a", "test-instance");
NetworkId networkId = NetworkId.of("default");
PersistentDiskConfiguration attachConfiguration =
    PersistentDiskConfiguration.builder(diskId).boot(true).build();
AttachedDisk attachedDisk = AttachedDisk.of("dev0", attachConfiguration);
NetworkInterface networkInterface = NetworkInterface.builder(networkId)
    .accessConfigurations(NetworkInterface.AccessConfig.builder()
        .name("external-nat")
        .build())
    .build();
MachineTypeId machineTypeId = MachineTypeId.of("us-central1-a", "n1-standard-1");
InstanceInfo instance =
    InstanceInfo.of(instanceId, machineTypeId, attachedDisk, networkInterface);
operation = compute.create(instance);
// Wait for operation to complete
operation = operation.waitFor();
if (operation.errors() == null) {
  System.out.println("Instance " + instanceId + " was successfully created");
} else {
  // inspect operation.errors()
  throw new RuntimeException("Instance creation failed");
}
```

I you haven't created a disk already, you can also create a VM instance and create a boot disk 
on the fly. When the following code, the instance's boot disk will be created along with the instance
itself:

```java
Address externalIp = compute.getAddress(addressId);
ImageId imageId = ImageId.of("debian-cloud", "debian-8-jessie-v20160329");
NetworkId networkId = NetworkId.of("default");
AttachedDisk attachedDisk =
    AttachedDisk.of(CreateDiskConfiguration.builder(imageId)
        .autoDelete(true)
        .build());
NetworkInterface networkInterface = NetworkInterface.builder(networkId)
    .accessConfigurations(AccessConfig.of(externalIp.address()))
    .build();
InstanceId instanceId = InstanceId.of("us-central1-a", "test-instance");
MachineTypeId machineTypeId = MachineTypeId.of("us-central1-a", "n1-standard-1");
InstanceInfo instance =
    InstanceInfo.of(instanceId, machineTypeId, attachedDisk, networkInterface);
Operation operation = compute.create(instance);
// Wait for operation to complete
operation = operation.waitFor();
if (operation.errors() == null) {
  System.out.println("Instance " + instanceId + " was successfully created");
} else {
  // inspect operation.errors()
  throw new RuntimeException("Instance creation failed");
}
```

## Do More

Other features are supported by `gcloud-java`: managing disk images, networks, subnetwors and more. Have a look at [Compute's examples](https://github.com/GoogleCloudPlatform/gcloud-java/tree/master/gcloud-java-examples/src/main/java/com/google/cloud/examples/compute)
for a more complete set of examples. Javadoc for the latest version is also available
[here](http://googlecloudplatform.github.io/gcloud-java/0.2.3/apidocs/index.html).