# IPC Plugin

## 🚀 Introduction
The IPC (Inter-Process Communication) plugin is designed specifically for Unreal Engine projects. Using shared memory technology, the plugin enables efficient data exchange, supports serialization and deserialization of various data types, and provides Blueprint and Python interfaces for developers to quickly implement complex communication functionality.

## 📦 Installation Guide
1. Place the `IPC` plugin folder into the `Plugins` directory of your Unreal Engine project.
2. Enable the plugin in the Unreal Engine editor.
3. Ensure Python support is enabled in the project settings (optional).
4. Restart the editor to load the plugin.

## 📘 Features Overview
### Features:
- **High-performance shared memory communication**: Supports efficient data exchange between processes or projects.
- **Built-in serialization and deserialization support**:
  - **Primitive types**: `int32`, `int64`, `float`, `double`
  - **Strings**: `FString`
  - **Structs**: `FVector`, `FRotator`, `FVector2D`, `FVector2f`, `FVector3f`, `FVector4f`
  - **Arrays**: `TArray<int32>`, `TArray<float>`, `TArray<FVector>`, `TArray<FVector2D>`, etc.
- **Supports multiple workflows**: Fully compatible with C++, Blueprint, and Python.
- **Example project**: Includes example Blueprints and Python scripts for quick start.

## 🔵 Blueprint Tutorial
### Writing to Shared Memory
1. Add the `WriteSharedMemory` node in Blueprint.
2. Set the shared memory name and size.
3. Call the node to write data.

### Reading from Shared Memory
1. Add the `ReadSharedMemory` node in Blueprint.
2. Set the shared memory name.
3. Call the node to read data.

### Notes
- **Read Order Consistency**: When reading shared memory data, the read order must match the write order to avoid parsing errors.

## 🧱 C++ Usage Examples
### Writing to Shared Memory
Use the `ASharedMemoryTestActor::WriteSharedMemory` method to write data to shared memory. This method calls `USharedMemoryManager::WriteMemory` to serialize and store data in the shared memory region.

#### Example Code
```cpp
void ASharedMemoryTestActor::WriteSharedMemory()
{
    TArray<uint8> Bytes;
    int32 Offset = 0;

    // Serialize data
    USharedMemoryLibrary::SerializeInt32(Bytes, Offset, 123);
    USharedMemoryLibrary::SerializeFString(Bytes, Offset, TEXT("Hello, I am C++ example!"));
    USharedMemoryLibrary::SerializeFVector(Bytes, Offset, FVector(1.f, 2.f, 3.f));

    // Initialize shared memory manager
    if (!SharedMemoryManager)
    {
        SharedMemoryManager = NewObject<USharedMemoryManager>(this, TEXT("SharedMemoryManager"));
        MemorySize = FMath::Max(Bytes.Num() + 4, MemorySize); // Reserve 4 bytes for storing total byte count
        SharedMemoryManager->MapNamedSharedMemoryRegion(MemoryName, MemorySize, true);
    }

    // Write to shared memory
    if (SharedMemoryManager && SharedMemoryManager->WriteMemory(Bytes))
    {
        UE_LOG(LogSharedMemoryTestActor, Warning, TEXT("Data successfully written to shared memory! Total bytes: %d"), SharedMemoryManager->GetMemorySize());
    }
    else
    {
        UE_LOG(LogSharedMemoryTestActor, Error, TEXT("Failed to write data to shared memory!"));
    }
}
```

### Reading from Shared Memory
Use the `ASharedMemoryTestActor::ReadSharedMemory` method to read data from shared memory. This method calls `USharedMemoryManager::ReadMemory` and uses `USharedMemoryLibrary` for deserialization.

#### Example Code
```cpp
void ASharedMemoryTestActor::ReadSharedMemory()
{
    TArray<uint8> Bytes;

    // Initialize shared memory manager
    if (!SharedMemoryManager)
    {
        SharedMemoryManager = NewObject<USharedMemoryManager>(this, TEXT("SharedMemoryManager"));
        SharedMemoryManager->MapNamedSharedMemoryRegion(MemoryName, MemorySize, false);
    }

    // Read from shared memory
    if (SharedMemoryManager->ReadMemory(Bytes))
    {
        int32 Offset = 0;

        // Deserialize data
        int32 IntValue = USharedMemoryLibrary::DeserializeInt32(Bytes, Offset);
        FString StringValue = USharedMemoryLibrary::DeserializeFString(Bytes, Offset);
        FVector VectorValue = USharedMemoryLibrary::DeserializeFVector(Bytes, Offset);

        // Print validation
        UE_LOG(LogSharedMemoryTestActor, Log, TEXT("Read Int32: %d"), IntValue);
        UE_LOG(LogSharedMemoryTestActor, Log, TEXT("Read FString: %s"), *StringValue);
        UE_LOG(LogSharedMemoryTestActor, Log, TEXT("Read FVector: %s"), *VectorValue.ToString());
    }
    else
    {
        UE_LOG(LogSharedMemoryTestActor, Error, TEXT("Failed to read shared memory!"));
    }
}
```

## 🐍 Python Usage Examples
### Writing to Shared Memory
Use the `example_write()` function in `unrealShm.py` to write data to shared memory. This function serializes various data types (including primitives, structs, and arrays) and calls `write_shared_memory` to store the data.

#### Example Code
```python
def example_write():
    writer = FMemoryWriter()

    # Write data
    writer.write_int32(123)
    writer.write_fstring("Hello, I am Python example!")
    FVector(1.1, 2.2, 3.3).serialize(writer)

    # Write to shared memory
    memory_name = hash_md5_name("UE5SharedMemory")
    memory_size = max(1024, len(writer.get_bytes()) + 4)
    success = write_shared_memory(memory_name, writer.get_bytes(), memory_size)

    logging.warning(
        "[ExampleWrite] Memory name: %s, Bytes written: %d, Result: %s",
        memory_name,
        len(writer.get_bytes()),
        "Success" if success else "Failed",
    )
```

### Reading from Shared Memory
Use the `example_read()` function in `unrealShm.py` to read data from shared memory. This function calls `read_shared_memory` to retrieve the data and uses `FMemoryReader` for deserialization.

#### Example Code
```python
def example_read():
    memory_name = hash_md5_name("UE5SharedMemory")
    raw_data = read_shared_memory(memory_name, 1024)

    if raw_data:
        reader = FMemoryReader(raw_data)
        int_value = reader.read_int32()
        string_value = reader.read_fstring()
        vector_value = FVector.deserialize(reader)

        logging.info("Read Int32: %d", int_value)
        logging.info("Read FString: %s", string_value)
        logging.info("Read FVector: %s", vector_value)
    else:
        logging.error("[ExampleRead] Failed to read shared memory or no data")
```

## 💡 Usage Scenarios
1. **Cross-process data exchange**: Efficiently exchange data between Unreal Engine and external applications (e.g., Python scripts, data processing tools).
2. **Real-time data sharing**: Share real-time data such as game state, sensor data, or external control signals.
3. **Multithreaded communication**: Use shared memory for thread-safe communication in multithreaded environments.
4. **Cross-platform integration**: Enable communication between different platforms, such as Windows and Linux.
5. **Extension Support**: Through extensions, the plugin can support sharing custom structs, images, meshes, and other complex data types.

### Extension Support Details
- **Custom Structs**: Developers can extend serialization and deserialization logic to support sharing custom struct data, such as character states or physical properties in games.
- **Image Data**: Supports sharing image data, such as textures or frame buffers. Developers can serialize pixel data and write it to shared memory for inter-process communication.
- **Mesh Data**: Supports sharing mesh data, such as vertices, normals, UV coordinates, etc. By extending serialization logic, mesh data can be written to shared memory and read by other processes.

## 📌 Best Practices and Notes
1. **Unique shared memory names**: Ensure each shared memory region has a unique name to avoid conflicts.
2. **Shared memory identifier**: Shared memory uses MD5 hash as a unique identifier to ensure security and uniqueness in cross-process communication.
3. **Shared memory size**:
   - When writing to shared memory, ensure the memory size is larger than the data size to be written; otherwise, the write operation will fail.
   - When reading from shared memory, ensure the read size is not smaller than the written data size; otherwise, the read data may be incomplete.
4. **Read order consistency**: When reading shared memory data, the read order must match the write order to avoid parsing errors.
5. **Thread safety**: Use `FCriticalSection` or other synchronization mechanisms to ensure thread safety when accessing shared memory in multithreaded environments.
6. **Resource release**: Call `CloseMemory` to release resources after completing shared memory operations.

## 📦 Plugin Information
### Technical Details
- **Supported Platforms**: Windows, Mac, Linux, Android, iOS
- **Number of Blueprints**: 40+
- **Number of C++ Classes**: 4
- **Network Replicated**: No

## 🛠️ Support
If you encounter any issues, bugs, or have feature requests, you can create a new issue on GitHub [here]([Issues · xusjtuer/FabTutorials](https://github.com/xusjtuer/FabTutorials/issues)). Providing replication steps, screenshots, or blueprintUE links for formatting issues would be greatly helpful.

For general questions about the plugin, you can ask them on the GitHub page or on the Unreal Engine forum.

For private inquiries, you can send an email to: xusjtuer@163.com.