# USB Mass Storage Plugin - Product Documentation

## Product Overview

The USB Mass Storage plugin is an enterprise-grade solution for RDK-based devices that provides seamless, automatic management of USB storage devices. Built on the WPEFramework (Thunder) platform, it delivers a robust, standardized API for detecting, mounting, and accessing external USB storage media, enabling applications to leverage removable storage for content delivery, local caching, recording, and data transfer scenarios.

## Key Features

### Automatic Device Management
- **Plug-and-Play Operation**: Automatically detects and mounts USB storage devices upon insertion without user intervention
- **Hot-Swap Support**: Safely handles device removal with automatic unmounting and cleanup
- **Multi-Partition Support**: Discovers and mounts all partitions on multi-partition USB devices
- **File System Compatibility**: Supports FAT32 (VFAT) and exFAT file systems commonly used on consumer USB devices

### Real-Time Notifications
- **Event-Driven Architecture**: Applications receive immediate notifications when devices are mounted or unmounted
- **Detailed Device Information**: Event payloads include device path, mount points, partition details, and file system type
- **Multiple Subscriber Support**: Allows multiple applications to register for storage events simultaneously

### Comprehensive Device Querying
- **Device Enumeration**: Query all currently connected USB storage devices
- **Mount Point Discovery**: Retrieve all active mount points for a specific device
- **Partition Statistics**: Access real-time information about partition size, usage, and available space

## Use Cases and Target Scenarios

### 1. **Local Content Storage and Playback**
Applications can leverage USB storage for:
- Storing downloaded video content for offline playback
- Caching media files to reduce network bandwidth consumption
- Loading local content libraries (photos, music, videos)

**Integration Benefit**: Automatic mount detection enables media players to immediately index and present USB content to users.

### 2. **DVR and Recording Applications**
Recording applications utilize USB storage for:
- Storing scheduled recordings when internal storage is limited
- Providing expandable storage capacity for long-term archival
- Enabling recording portability across devices

**Integration Benefit**: Real-time space monitoring through partition info API ensures recordings don't fail due to insufficient storage.

### 3. **Firmware and Software Updates**
System update mechanisms can use USB storage for:
- Manual firmware image deployment in development environments
- Offline update delivery in environments without network connectivity
- Emergency recovery and diagnostics tool loading

**Integration Benefit**: Standardized device discovery eliminates custom USB handling code in update managers.

### 4. **Diagnostic Log Collection**
Support and diagnostics tools leverage USB storage for:
- Exporting system logs for offline analysis
- Capturing crash dumps and debugging information
- Transferring diagnostic data without network dependencies

**Integration Benefit**: Automatic mount notifications trigger log export workflows without manual intervention.

### 5. **Data Transfer and File Sharing**
General-purpose applications use USB storage for:
- Transferring user files between devices
- Importing configuration data
- Exporting user-generated content

**Integration Benefit**: Unified API eliminates platform-specific USB handling across application portfolio.

## API Capabilities and Integration Benefits

### JSON-RPC Interface

The plugin exposes a Thunder JSON-RPC interface accessible via WebSocket or HTTP transport:

**Endpoint Pattern**: `Controller.1.activate?callsign=org.rdk.UsbMassStorage`

#### Core Methods

1. **getDeviceList**
   - **Purpose**: Enumerate all connected USB storage devices
   - **Returns**: Array of device objects with device path, vendor ID, product ID
   - **Use Case**: UI display of available storage options

2. **getMountPoints**
   - **Purpose**: Retrieve mount paths for a specific device
   - **Parameters**: Device name (e.g., "sda")
   - **Returns**: Array of mount point strings
   - **Use Case**: Accessing file content on specific partitions

3. **getPartitionInfo**
   - **Purpose**: Get storage statistics for a mount point
   - **Parameters**: Mount path
   - **Returns**: Total size, used space, available space, file system type
   - **Use Case**: Pre-flight checks before writing large files

#### Event Notifications

1. **onDeviceMounted**
   - **Triggered**: When USB device partition successfully mounts
   - **Payload**: Device info, mount path, partition number, file system
   - **Use Case**: Trigger content indexing or user notification

2. **onDeviceUnmounted**
   - **Triggered**: When USB device is removed or unmounted
   - **Payload**: Device info, previously mounted path
   - **Use Case**: Cleanup cached references, update UI

### Integration Advantages

#### Simplified Application Development
- **No Direct System Calls**: Applications avoid complex mount/umount operations
- **Cross-Platform Consistency**: Same API across different RDK device types
- **Error Handling**: Centralized error management and logging

#### Operational Reliability
- **Safe Unmounting**: Prevents data corruption through proper unmount sequencing
- **Resource Cleanup**: Automatic cleanup of mount points on device removal
- **Conflict Prevention**: Manages concurrent access to USB devices

#### Security and Isolation
- **Controlled Access**: Applications access storage through standardized API
- **Permission Management**: Integration with Thunder's security token system
- **Audit Trail**: Centralized logging of all USB storage operations

## Performance and Reliability Characteristics

### Performance Metrics

- **Detection Latency**: Typically <500ms from physical insertion to mount completion
- **Event Propagation**: <100ms from mount completion to notification delivery
- **Query Response**: <50ms for device list and partition info queries
- **Concurrent Clients**: Supports 10+ simultaneous notification subscribers without degradation

### Reliability Features

#### Fault Tolerance
- **Corrupted File Systems**: Gracefully handles mount failures with detailed error logging
- **Abrupt Removal**: Safely handles device removal during active I/O
- **System Recovery**: Automatically re-mounts devices after system restart (boot-up detection)

#### Resource Management
- **Memory Efficiency**: Minimal heap allocation with COM reference counting
- **Thread Safety**: All operations protected by critical sections
- **Mount Point Isolation**: Uses `/tmp/media/usb` hierarchy to avoid conflicts

#### Error Recovery
- **Mount Retry**: Automatic retry logic for transient mount failures
- **Cleanup Guarantee**: Ensures mount directories are removed even on error paths
- **Notification Resilience**: Continues operation even if notification subscribers fail

### Quality Assurance

- **L1 Unit Tests**: Comprehensive testing of core functionality with mocked system calls
- **L2 Integration Tests**: End-to-end testing with real USB device scenarios
- **Coverage Reporting**: Code coverage tracking with lcov
- **Continuous Integration**: Automated build and test pipeline via GitHub Actions

## System Requirements

### Runtime Dependencies
- WPEFramework (Thunder) R4.4.1 or later
- libusb-1.0 for USB device enumeration
- Linux kernel with USB mass storage and file system support (VFAT, exFAT)

### Installation
- Plugin automatically installed to `/usr/lib/wpeframework/plugins`
- Configuration file located at `/etc/entservices/USBMassStorage.config`
- Requires `org.rdk.UsbDevice` plugin to be installed and activated

### Resource Footprint
- Binary Size: ~100KB (shared library)
- Memory Usage: <2MB runtime footprint
- CPU Usage: Negligible except during mount operations (<1% peak)

## Migration and Compatibility

### Legacy Integration
For systems transitioning from custom USB handling:
- Provides backward-compatible notification patterns
- Supports gradual migration of USB-dependent applications
- Coexists with existing file system access patterns

### Future Roadmap
- Support for additional file systems (NTFS, ext4)
- USB device allowlist/blocklist configuration
- Enhanced security policies for storage access control
- Performance optimizations for high-speed USB 3.x devices
