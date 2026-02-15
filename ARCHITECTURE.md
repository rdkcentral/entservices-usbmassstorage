# USB Mass Storage Plugin - Architecture Documentation

## Overview

The USB Mass Storage plugin is a WPEFramework (Thunder) plugin that provides automatic USB mass storage device detection, mounting, and management capabilities for RDK devices. The plugin integrates with the Thunder framework to expose USB storage functionality through a standardized JSON-RPC interface.

## System Architecture

### Component Hierarchy

```
┌─────────────────────────────────────────────────────┐
│         WPEFramework (Thunder)                      │
│         Plugin Host & JSON-RPC Layer                │
└─────────────────┬───────────────────────────────────┘
                  │
                  ├──> USBMassStorage Plugin (JSON-RPC Interface)
                  │    - Register/Unregister notifications
                  │    - GetDeviceList, GetMountPoints
                  │    - GetPartitionInfo
                  │
                  └──> USBMassStorageImplementation (Core Logic)
                       - USB device event handling
                       - Mount/unmount operations
                       - File system detection
                       - Partition management
                       │
                       └──> USBDevice Plugin Interface
                            - Device plug/unplug notifications
                            - USB device enumeration
```

### Component Interactions

#### 1. **USBMassStorage Plugin Layer**
   - **Role**: JSON-RPC interface wrapper
   - **Responsibilities**:
     - Exposes Thunder plugin APIs
     - Handles JSON-RPC request/response serialization
     - Manages plugin lifecycle (Initialize, Deinitialize)
     - Registers notification callbacks with WPEFramework

#### 2. **USBMassStorageImplementation**
   - **Role**: Core business logic implementation
   - **Responsibilities**:
     - Implements `IUSBMassStorage` and `IConfiguration` interfaces
     - Registers with USBDevice plugin for device events
     - Handles USB mass storage device mounting/unmounting
     - Manages device and mount point tracking
     - Dispatches storage events to registered clients
     - Detects file system types (VFAT, exFAT)

#### 3. **USBDevice Plugin Integration**
   - **Role**: Underlying USB hardware abstraction
   - **Responsibilities**:
     - Detects USB device plug/unplug events via libusb
     - Provides device enumeration
     - Filters mass storage class devices
     - Notifies USBMassStorage of relevant events

## Data Flow

### Device Mount Sequence

1. **Device Detection**
   - USBDevice plugin detects new USB device via libusb
   - Identifies device as mass storage class
   - Fires `OnDevicePluggedIn` notification

2. **Storage Processing**
   - USBMassStorageImplementation receives notification
   - Reads `/proc/partitions` to discover partitions
   - Iterates through partitions on the device

3. **Mounting**
   - Creates mount directory at `/tmp/media/usb/{partition}`
   - Detects file system type using `statfs`
   - Executes mount system call with appropriate options
   - Updates internal mount registry

4. **Client Notification**
   - Dispatches mount event to registered notification callbacks
   - Provides mount path, device info, and partition details

### Device Unmount Sequence

1. **Device Removal**
   - USBDevice plugin detects device removal
   - Fires `OnDevicePluggedOut` notification

2. **Unmount Processing**
   - Locates all mount points for device
   - Executes `umount` for each partition
   - Removes mount directories
   - Updates internal registries

3. **Client Notification**
   - Dispatches unmount events to registered callbacks

## Plugin Framework Integration

### Thunder Plugin Model

The plugin follows WPEFramework's standard plugin architecture:

- **Service Registration**: `SERVICE_REGISTRATION(USBMassStorage, 1, 0, 0)`
- **Metadata Declaration**: Version, preconditions, and controls
- **Interface Implementation**: COM-style interfaces with `BEGIN_INTERFACE_MAP`

### Configuration

- **Auto-start**: Configurable via `PLUGIN_USB_MASS_STORAGE_AUTOSTART`
- **Startup Order**: Configurable via `PLUGIN_USB_MASS_STORAGE_STARTUPORDER` (default: 45)
- **Configuration File**: `USBMassStorage.config` for runtime settings

### Threading Model

- **Event Dispatching**: Uses WPEFramework's `Core::IDispatch` job mechanism
- **Thread Safety**: Critical sections protect shared device/mount registries
- **Asynchronous Notifications**: Mount/unmount operations dispatch events asynchronously

## Dependencies and Interfaces

### External Dependencies

- **WPEFramework Core**: COM runtime, threading, IPC
- **WPEFramework Plugins**: Plugin host infrastructure
- **libusb-1.0**: USB device enumeration and management
- **System Libraries**: mount, umount, statfs syscalls

### Interface Definitions

1. **IUSBMassStorage** (entservices-apis)
   - `Register/Unregister`: Notification subscription
   - `GetDeviceList`: Enumerate connected storage devices
   - `GetMountPoints`: Query mount points for a device
   - `GetPartitionInfo`: Retrieve partition size/usage statistics

2. **IUSBDevice** (entservices-apis)
   - `INotification`: Device plug/unplug callbacks
   - Device information structures

### Helper Utilities

- **UtilsLogging.h**: Logging macros (LOGINFO, LOGERR, etc.)
- **UtilsJsonRpc.h**: JSON-RPC helper macros for parameter validation

## Technical Implementation Details

### File System Support

- **VFAT**: FAT32 file systems (type: 0x4d44)
- **exFAT**: Extended FAT (type: 0x2011)
- **Detection**: Uses `statfs.f_type` magic numbers
- **Mount Options**: Read-write by default, with error handling

### Mount Path Convention

- **Base Path**: `/tmp/media`
- **Mount Pattern**: `/tmp/media/usb/{device}{partition_number}`
- **Example**: `/tmp/media/usb/sda1`

### Error Handling

- **Mount Failures**: Logged with errno details
- **Device Conflicts**: Prevents duplicate device processing
- **Notification Safety**: Validates callback registration to prevent duplicates

### Memory Management

- **Reference Counting**: COM-style AddRef/Release for all interfaces
- **Smart Pointers**: `Core::ProxyType` for job dispatching
- **RAII**: Ensures proper cleanup on exceptions

## Build System

- **CMake-based**: Modern CMake 3.3+ required
- **Build Flags**: Configurable L1/L2 test support
- **Link Options**: Supports function wrapping for test mocking
- **Installation**: Plugins installed to `lib/{namespace}/plugins`

## Testing Infrastructure

- **L1 Tests**: Unit tests for core functionality
- **L2 Tests**: Integration tests with mock system calls
- **Mock Support**: Wraps mount, umount, ioctl, statfs for testing
- **Test Framework**: CUnit-based with coverage reporting
