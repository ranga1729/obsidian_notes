Software Defined Networks (SDN) represent an approach to networking that fundamentally changes how network control and data forwarding functions are managed. The core concept behind SDN is the **separation of the control plane from the data plane**. This means that the network control software, which determines optimal paths and responds to network demands, is removed from individual networking devices and placed into a centralized computational resource.

Here are the key concepts and aspects of Software Defined Networks:

### Evolution Leading to SDN

Traditionally, switching functions were integrated within network devices and categorized into three planes: data, control, and management.

- **Data Plane**: Handles the physical reception and transmission of packets, including buffering, scheduling, header modification, and forwarding. If a packet's header information is in the forwarding table, it's processed and forwarded without intervention from other planes. Otherwise, it's sent to the control plane.
- **Control Plane**: Primarily responsible for keeping the forwarding table updated. It processes various control protocols that manage the network's active topology, requiring general-purpose microprocessors and software within the device.
- **Management Plane**: Used by network administrators to configure and monitor the network, extracting or modifying data in the control and data planes using a network management system.

In early networking, most functions, except the physical layer, were software-controlled within bridges, switches, and routers. Over time, basic functions like forwarding and filtering decisions transitioned from software to hardware for efficiency. However, complex control software for routing decisions and interacting with other devices to converge on topologies and paths remained embedded in software running autonomously on each device.

**SDN's key shift** is to **move this complex control software off the individual network device** and into a centralized controller. This controller gains a complete view of the entire network, enabling it to make global, optimal decisions. This separation allows for:

- **Forwarding, Filtering, and Prioritization**: These responsibilities, implemented in hardware tables, remain on the network device.
- **Centralized Control**: The complicated control software is now located in a centralized controller.
- **Applications**: Network applications run above this centralized controller.

### Fundamental Characteristics of SDN

SDN is defined by several core characteristics:

- **Plane Separation**: The explicit separation of the forwarding (data) and control planes.
- **Simplified Device and Centralized Control**: Network devices become simpler as management and control software is centralized, reducing complexity on individual devices.
- **Network Automation and Virtualization**: Enables automated configuration and dynamic management of network resources.
- **Distributed State Abstraction**: Provides a global, holistic view of the network.
- **Forwarding Abstraction**: Allows specifying forwarding behaviors without needing to know vendor-specific hardware details.
- **Configuration Abstraction**: Defines network goals without detailing how they are implemented in the physical hardware.
- **Openness**: Open SDN refers to standards-based, well-documented, non-proprietary solutions.

### Main Components of SDN Architecture

An SDN architecture typically comprises three main components:

- **SDN Devices**: These are the physical or virtual network elements (like switches) that compose the data plane. They include an API for communication with the controller, an abstraction layer, and a packet processing function.
- **SDN Controller**: This is the centralized software-based brain of the SDN. It maintains a view of the entire network, implements policy decisions, controls all SDN devices, and provides a **Northbound API**. The Northbound API allows software applications to connect to the controller, providing algorithms and protocols to run the network efficiently.
- **SDN Applications**: These applications run above the SDN controller and interface with the network via the controller's Northbound API.

### OpenFlow

**OpenFlow is one of the pivotal technologies used for SDN**. It defines both the communication protocol between the SDN data plane (switches) and the SDN control plane (controller), but it does not describe the controller's internal behavior. An OpenFlow system consists of:

- **OpenFlow Controller**: Communicates with one or more OpenFlow Switches.
- **OpenFlow Protocol**: Defines the specific messages and message formats exchanged between the controller and the device.
- **OpenFlow Behavior**: Specifies how the device should react in various situations and how it should respond to commands from the controller.

### SDN Use Cases

SDN technologies are being explored and implemented in various real-world scenarios, particularly in data centers, to orchestrate and manage networks more flexibly. Examples include Google implementing Open SDN with lightweight OpenFlow switches and controllers for managing WAN connections between their data centers, and Microsoft Azure using overlay technology with NVGRE in vSwitches for creating virtual networks.