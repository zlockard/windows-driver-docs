---
title: Starting a BDA Minidriver
author: windows-driver-content
description: Starting a BDA Minidriver
MS-HAID:
- 'bdadg\_26ed5e0c-c521-4011-8a2b-2e51b05dc25d.xml'
- 'stream.starting\_a\_bda\_minidriver'
MSHAttr:
- 'PreferredSiteName:MSDN'
- 'PreferredLib:/library/windows/hardware'
ms.assetid: c71e1483-756c-4e98-a413-64ff02ee4a9b
keywords: ["BDA minidrivers WDK AVStream , starting", "starting BDA minidrivers WDK AVStream"]
---

# Starting a BDA Minidriver


## <a href="" id="ddk-starting-a-bda-minidriver-ksg"></a>


When a BDA device starts operating, the Plug and Play (PnP) manager dispatches [**IRP\_MN\_START\_DEVICE**](https://msdn.microsoft.com/library/windows/hardware/ff551749). The AVStream class in turn calls the start routine of the BDA minidriver associated with the BDA device. This start routine retrieves information about the device from the registry, sets information about the device, and then calls the [**BdaCreateFilterFactory**](https://msdn.microsoft.com/library/windows/hardware/ff556438) support function to:

-   Create the filter factory for the device from the initial filter descriptor ([**KSFILTER\_DESCRIPTOR**](https://msdn.microsoft.com/library/windows/hardware/ff562553)) for the device. The initial filter descriptor references dispatch and automation tables for the filter and input pins. See [Creating Dispatch Tables](creating-dispatch-tables.md) and [Defining Automation Tables](defining-automation-tables.md) for more information.

-   Associate the filter factory with a [**BDA\_FILTER\_TEMPLATE**](https://msdn.microsoft.com/library/windows/hardware/ff556523) structure. This structure references the template filter descriptor for the device and the list of possible pairs of input and output pins. This descriptor and list in turn reference:
    -   Static template structures that a network provider can use to determine the topology of a BDA filter.
    -   Nodes and pins for a BDA filter along with possible ways to connect the filter.
    -   Routines that a network provider can use to create and close a filter instance.
    -   Static template structures that a network provider can use to manipulate a BDA filter.
-   Register the static template structures that are specified by BDA\_FILTER\_TEMPLATE with the BDA support library so that the library can provide default handling for a BDA minidriver's properties and methods.

The following code snippet shows an example of an initial filter descriptor for the device that **BdaCreateFilterFactory** sets as the filter factory:

```
const KSFILTER_DESCRIPTOR    InitialTunerFilterDescriptor;
//
//  Filter Factory Descriptor for the tuner filter
//
//  This structure brings together all of the structures that define
//  the tuner filter instance as it appears when it is first created.
//  Note that not all template pin and node types are exposed as
//  pin and node factories when the filter instance is created.
//
DEFINE_KSFILTER_DESCRIPTOR(InitialTunerFilterDescriptor)
{
    &amp;FilterDispatch,             // Table of dispatch routines
    &amp;FilterAutomation,           // Table of properties and methods
    KSFILTER_DESCRIPTOR_VERSION, // Version
    0,                           // Flags
    &amp;KSNAME_Filter,              // Reference Guid
    DEFINE_KSFILTER_PIN_DESCRIPTORS(InitialPinDescriptors),
                                   // PinDescriptorsCount
                                   // PinDescriptorSize
                                   // PinDescriptors
    DEFINE_KSFILTER_CATEGORY(KSCATEGORY_BDA_RECEIVER_COMPONENT),
                            // CategoriesCount
                            // Categories
    DEFINE_KSFILTER_NODE_DESCRIPTORS_NULL(NodeDescriptors),
                                    // NodeDescriptorsCount
                                    // NodeDescriptorSize
                                    // NodeDescriptors
    DEFINE_KSFILTER_DEFAULT_CONNECTIONS, // ConnectionsCount
                                         // Connections
    NULL                // ComponentId
};
```

The following code snippet shows an example of an array of initial pin descriptors that are exposed by an initialized filter. The network provider initializes a filter using such an array before the network provider configures that filter. However, when configuring an initialized filter, the network provider selects pins that are referenced in the pointer to the filter descriptor member of the BDA\_FILTER\_TEMPLATE structure. See [Configuring a BDA Filter](configuring-a-bda-filter.md) for more information.

```
//
//  Initial Pin Descriptors
//
//  This data structure defines the pins that will appear on the 
//  filter when it is first created.
//
const
KSPIN_DESCRIPTOR_EX
InitialPinDescriptors[] =
{
    //  Antenna Pin
    //
    {
        &amp;AntennaPinDispatch,
        &amp;AntennaAutomation,   // AntennaPinAutomation
        {
            0,  // Interfaces
            NULL,
            0,  // Mediums
            NULL,
            SIZEOF_ARRAY(AntennaPinRanges),
            AntennaPinRanges,
            KSPIN_DATAFLOW_IN,
            KSPIN_COMMUNICATION_BOTH,
            NULL,   // Name
            NULL,   // Category
            0
        },
        KSPIN_FLAG_DO_NOT_USE_STANDARD_TRANSPORT | 
        KSPIN_FLAG_FRAMES_NOT_REQUIRED_FOR_PROCESSING | 
        KSPIN_FLAG_FIXED_FORMAT,
        1,      // InstancesPossible
        0,      // InstancesNecessary
        NULL,   // Allocator Framing
        NULL    // PinIntersectHandler
    }
};
```

Note that an initialized filter must have one or more input pins exposed so that the Microsoft DirectShow **IFilterMapper2** or **IFilterMapper** interface can locate that filter. See the Microsoft Windows SDK documentation for information about these DirectShow interfaces.

The following code snippet shows examples of a BDA\_FILTER\_TEMPLATE structure and related structures and arrays:

```
const KSFILTER_DESCRIPTOR  TemplateTunerFilterDescriptor;
const BDA_PIN_PAIRING  *TemplateTunerPinPairings;
//
//  BDA Template Topology Descriptor for the filter factory.
//
//  This structure defines the pin and node types that the network 
//  provider can create on the filter and how they are arranged. 
//
const
BDA_FILTER_TEMPLATE
TunerBdaFilterTemplate =
{
    &amp;TemplateTunerFilterDescriptor,
    SIZEOF_ARRAY(TemplateTunerPinPairings),
    TemplateTunerPinPairings
};
//
//  Filter Factory Descriptor for the tuner filter template topology
//
//  This structure brings together all of the structures that define
//  the topologies that the tuner filter can assume as a result of
//  pin factory and topology creation methods.
//
DEFINE_KSFILTER_DESCRIPTOR(TemplateTunerFilterDescriptor)
{
    &amp;FilterDispatch,             // Table of dispatch routines
    &amp;FilterAutomation,           // Table of properties and methods
    KSFILTER_DESCRIPTOR_VERSION, // Version
    0,                           // Flags
    &amp;KSNAME_Filter,              // Reference Guid
    DEFINE_KSFILTER_PIN_DESCRIPTORS(TemplatePinDescriptors),
                                   // PinDescriptorsCount
                                   // PinDescriptorSize
                                   // PinDescriptors
    DEFINE_KSFILTER_CATEGORY(KSCATEGORY_BDA_RECEIVER_COMPONENT),
                            // CategoriesCount
                            // Categories
    DEFINE_KSFILTER_NODE_DESCRIPTORS(NodeDescriptors),  
                                    // NodeDescriptorsCount
                                    // NodeDescriptorSize
                                    // NodeDescriptors
    DEFINE_KSFILTER_CONNECTIONS(TemplateTunerConnections),
                               // ConnectionsCount
                               // Connections
    NULL                // ComponentId
};
//
//  Lists how pairs of input and output pins are configured. 
//
//  Values given to the BDA_PIN_PAIRING structures in the list inform 
//  the network provider which nodes get duplicated when more than one 
//  output pin type is connected to a single input pin type or when
//  more that one input pin type is connected to a single output pin 
//  type. Also, informs of the joints array.
//
const
BDA_PIN_PAIRING TemplateTunerPinPairings[] =
{
    //  Antenna to Transport Topology Joints
    //
    {
        0,  // ulInputPin
        1,  // ulOutputPin
        1,  // ulcMaxInputsPerOutput
        1,  // ulcMinInputsPerOutput
        1,  // ulcMaxOutputsPerInput
        1,  // ulcMinOutputsPerInput
        SIZEOF_ARRAY(AntennaTransportJoints),   // ulcTopologyJoints
        AntennaTransportJoints                  // pTopologyJoints
    }
};
```

 

 


--------------------
[Send comments about this topic to Microsoft](mailto:wsddocfb@microsoft.com?subject=Documentation%20feedback%20%5Bstream\stream%5D:%20Starting%20a%20BDA%20Minidriver%20%20RELEASE:%20%288/23/2016%29&body=%0A%0APRIVACY%20STATEMENT%0A%0AWe%20use%20your%20feedback%20to%20improve%20the%20documentation.%20We%20don't%20use%20your%20email%20address%20for%20any%20other%20purpose,%20and%20we'll%20remove%20your%20email%20address%20from%20our%20system%20after%20the%20issue%20that%20you're%20reporting%20is%20fixed.%20While%20we're%20working%20to%20fix%20this%20issue,%20we%20might%20send%20you%20an%20email%20message%20to%20ask%20for%20more%20info.%20Later,%20we%20might%20also%20send%20you%20an%20email%20message%20to%20let%20you%20know%20that%20we've%20addressed%20your%20feedback.%0A%0AFor%20more%20info%20about%20Microsoft's%20privacy%20policy,%20see%20http://privacy.microsoft.com/default.aspx. "Send comments about this topic to Microsoft")


