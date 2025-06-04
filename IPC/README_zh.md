# IPC 插件

## 🚀 插件简介
IPC（进程间通信）插件是一款专为 Unreal Engine 项目设计的解决方案。通过共享内存技术，插件实现了高效的数据交换，支持多种数据类型的序列化与反序列化，并提供蓝图和 Python 接口，方便开发者快速实现复杂的通信功能。

## 📦 安装指南
1. 将插件文件夹 `IPC` 放置到 Unreal Engine 项目的 `Plugins` 目录下。
2. 在 Unreal Engine 编辑器中启用插件。
3. 确保项目设置中启用了 Python 支持（可选）。
4. 重启编辑器以加载插件。

## 📘 功能概览
### 特性:
- **高性能共享内存通信**：支持进程间或项目间的高效数据交换。
- **内置序列化与反序列化支持**：
  - **基本类型**：`int32`, `int64`, `float`, `double`
  - **字符串**：`FString`
  - **结构体**：`FVector`, `FRotator`, `FVector2D`, `FVector2f`, `FVector3f`, `FVector4f`
  - **数组**：`TArray<int32>`, `TArray<float>`, `TArray<FVector>`, `TArray<FVector2D>` 等
- **支持多种工作流**：全面支持 C++、蓝图和 Python。
- **示例项目**：包含示例蓝图和 Python 脚本，方便快速上手。

## 🔵 蓝图教程
### 写入共享内存
1. 在蓝图中添加 `WriteSharedMemory` 节点。
2. 设置共享内存名称和大小。
3. 调用节点写入数据。

### 读取共享内存
1. 在蓝图中添加 `ReadSharedMemory` 节点。
2. 设置共享内存名称。
3. 调用节点读取数据。

### 注意事项
- **读取次序一致性**：读取共享内存数据时，读取的次序必须和写入的保持一致，否则可能导致数据解析异常。

## 🧱 C++ 使用案例
### 写入共享内存
使用 `ASharedMemoryTestActor::WriteSharedMemory` 方法将数据写入共享内存。此方法会调用 `USharedMemoryManager::WriteMemory`，将序列化后的数据写入共享内存区域。

#### 示例代码
```cpp
void ASharedMemoryTestActor::WriteSharedMemory()
{
    TArray<uint8> Bytes;
    int32 Offset = 0;

    // 序列化数据
    USharedMemoryLibrary::SerializeInt32(Bytes, Offset, 123);
    USharedMemoryLibrary::SerializeFString(Bytes, Offset, TEXT("你好，I am c++ example ！"));
    USharedMemoryLibrary::SerializeFVector(Bytes, Offset, FVector(1.f, 2.f, 3.f));

    // 初始化共享内存管理器
    if (!SharedMemoryManager)
    {
        SharedMemoryManager = NewObject<USharedMemoryManager>(this, TEXT("SharedMemoryManager"));
        MemorySize = FMath::Max(Bytes.Num() + 4, MemorySize); // 保留4字节用于存储总字节数
        SharedMemoryManager->MapNamedSharedMemoryRegion(MemoryName, MemorySize, true);
    }

    // 写入共享内存
    if (SharedMemoryManager && SharedMemoryManager->WriteMemory(Bytes))
    {
        UE_LOG(LogSharedMemoryTestActor, Warning, TEXT("数据成功写入共享内存！总字节数：%d"), SharedMemoryManager->GetMemorySize());
    }
    else
    {
        UE_LOG(LogSharedMemoryTestActor, Error, TEXT("写入共享内存失败！"));
    }
}
```

### 读取共享内存
使用 `ASharedMemoryTestActor::ReadSharedMemory` 方法从共享内存读取数据。此方法会调用 `USharedMemoryManager::ReadMemory`，并使用 `USharedMemoryLibrary` 提供的反序列化方法解析数据。

#### 示例代码
```cpp
void ASharedMemoryTestActor::ReadSharedMemory()
{
    TArray<uint8> Bytes;

    // 初始化共享内存管理器
    if (!SharedMemoryManager)
    {
        SharedMemoryManager = NewObject<USharedMemoryManager>(this, TEXT("SharedMemoryManager"));
        SharedMemoryManager->MapNamedSharedMemoryRegion(MemoryName, MemorySize, false);
    }

    // 从共享内存读取数据
    if (SharedMemoryManager->ReadMemory(Bytes))
    {
        int32 Offset = 0;

        // 反序列化数据
        int32 IntValue = USharedMemoryLibrary::DeserializeInt32(Bytes, Offset);
        FString StringValue = USharedMemoryLibrary::DeserializeFString(Bytes, Offset);
        FVector VectorValue = USharedMemoryLibrary::DeserializeFVector(Bytes, Offset);

        // 打印验证
        UE_LOG(LogSharedMemoryTestActor, Log, TEXT("读取 Int32: %d"), IntValue);
        UE_LOG(LogSharedMemoryTestActor, Log, TEXT("读取 FString: %s"), *StringValue);
        UE_LOG(LogSharedMemoryTestActor, Log, TEXT("读取 FVector: %s"), *VectorValue.ToString());
    }
    else
    {
        UE_LOG(LogSharedMemoryTestActor, Error, TEXT("读取共享内存失败！"));
    }
}
```

## 🐍 Python 使用案例
### 写入共享内存
使用 `unrealShm.py` 中的 `example_write()` 方法将数据写入共享内存。此方法会序列化各种数据类型（包括基本类型、结构体和数组），并调用 `write_shared_memory` 将数据写入共享内存。

#### 示例代码
```python
def example_write():
    writer = FMemoryWriter()

    # 写入数据
    writer.write_int32(123)
    writer.write_fstring("你好，I am Python example !")
    FVector(1.1, 2.2, 3.3).serialize(writer)

    # 写入共享内存
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

### 读取共享内存
使用 `unrealShm.py` 中的 `example_read()` 方法从共享内存读取数据。此方法会调用 `read_shared_memory` 读取数据，并使用 `FMemoryReader` 提供的反序列化方法解析数据。

#### 示例代码
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

## 💡 使用场景示例
1. **跨进程数据交换**：在 Unreal Engine 和外部应用（如 Python 脚本、数据处理工具）之间进行高效的数据交换。
2. **实时数据共享**：实现实时数据共享，例如游戏状态、传感器数据或外部控制信号。
3. **多线程通信**：在多线程环境中使用共享内存进行线程间通信，确保数据一致性。
4. **跨平台集成**：通过共享内存实现不同平台之间的通信，例如 Windows 和 Linux 系统。
5. **扩展支持**：通过扩展，可以支持直接共享自定义结构体、图像、网格等复杂数据类型。

### 扩展支持说明
- **自定义结构体**：开发者可以通过扩展序列化和反序列化逻辑，支持共享自定义的结构体数据。例如，游戏中的角色状态、物理属性等。
- **图像数据**：支持共享图像数据，例如纹理或帧缓冲区。可以通过序列化图像的像素数据并写入共享内存。
- **网格数据**：支持共享网格数据，例如顶点、法线、UV 坐标等。可以通过扩展序列化逻辑，将网格数据写入共享内存并在其他进程中读取。

## 📌 注意事项与最佳实践
1. **共享内存名称唯一性**：确保每个共享内存区域的名称是唯一的，以避免冲突。
2. **共享内存标识符**：共享内存使用 MD5 哈希值作为唯一标识符，确保跨进程通信的安全性和唯一性。
3. **共享内存空间大小**：
   - 写入共享内存时，确保共享内存的空间大于待写入的数据大小，否则写入操作会失败。
   - 读取共享内存时，读取的空间大小必须不小于写入的数据大小，否则读取的数据可能不完整。
4. **读取次序一致性**：读取共享内存数据时，读取的次序必须和写入的保持一致，否则可能导致数据解析异常。
5. **线程安全**：在多线程环境中使用共享内存时，请使用 `FCriticalSection` 或其他同步机制确保线程安全。
6. **资源释放**：在完成共享内存操作后，调用 `CloseMemory` 方法释放资源。

## 📦 插件信息
### 技术详情
- **支持平台**：Windows、Mac、Linux、Android、iOS
- **蓝图数量**：40+
- **C++ 类数量**：4
- **网络同步**：不支持
- **文档链接**：[Documentation](https://example.com/documentation)
- **示例项目**：[Example Project](https://example.com/example-project)

## 🛠️ 支持
如果您遇到任何问题、错误或有功能请求，可以在 GitHub 上创建一个新问题：[点击这里](https://github.com/xusjtuer/IPC/issues)。提供复现步骤、截图或 blueprintUE 链接将非常有帮助。

对于插件的常见问题，您可以在 GitHub 页面或 Unreal Engine 论坛上提问。

如果您有关于插件的私人问题，可以发送电子邮件至：xusjtuer@163.com。