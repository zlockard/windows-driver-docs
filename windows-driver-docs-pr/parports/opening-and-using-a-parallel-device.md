---
title: Opening and Using a Parallel Device
author: windows-driver-content
description: Opening and Using a Parallel Device
MS-HAID:
- 'vspd\_65f06290-197a-4f32-8e23-512157432be4.xml'
- 'parports.opening\_and\_using\_a\_parallel\_device'
MSHAttr:
- 'PreferredSiteName:MSDN'
- 'PreferredLib:/library/windows/hardware'
ms.assetid: ca58b1c3-9ecf-4ebe-8f08-a2f78ae17921
keywords: ["parallel devices WDK , opening", "parallel devices WDK , sharing"]
---

# Opening and Using a Parallel Device


## <a href="" id="ddk-opening-and-using-a-parallel-device-kg"></a>


The system-supplied bus driver for parallel ports enforces exclusive access to a parallel device attached to a parallel port. If a parallel device is open, the parallel port bus driver fails any subsequent [**IRP\_MJ\_CREATE**](https://msdn.microsoft.com/library/windows/hardware/ff544131) requests for the device until the device has been closed. A client must open a parallel device before it sends other I/O requests to the device or calls the [parallel device callback routines](https://msdn.microsoft.com/library/windows/hardware/ff544275). A client must not attempt to communicate with a parallel device after the client has closed its file on a device. A client must close a device to allow other clients to access the device.

A client usually does the following:

-   Opens a parallel device

-   Connects to a parallel device − see [Connecting to a Parallel Device](connecting-to-a-parallel-device.md)

-   Obtains information about the parallel device − see [Obtaining Information about a Parallel Device](obtaining-information-about-a-parallel-device.md)

-   Locks the device − see [Locking and Unlocking a Parallel Port for Use by a Parallel Device](locking-and-unlocking-a-parallel-port-for-use-by-a-parallel-device.md)

-   Does a sequence of operations on the device

-   Disconnects from a parallel device − see [Connecting to a Parallel Device](connecting-to-a-parallel-device.md)

-   Unlocks the device − see [Locking and Unlocking a Parallel Port for Use by a Parallel Device](locking-and-unlocking-a-parallel-port-for-use-by-a-parallel-device.md)

-   Closes the device

Note that in a Plug and Play environment, a device can be removed or added whenever there are no open files on it. In general, every time a parallel device is added, Plug and Play assigns a different location and resources.

 

 


--------------------


