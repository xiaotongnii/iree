---
hide:
  - tags
tags:
  - CPU
icon: octicons/cpu-16
---

# CPU 部署

IREE 使用 [LLVM](https://llvm.org/) 将程序中的所有稠密计算编译为经过高度优化的 CPU 原生指令流，并将这些指令流嵌入 IREE 的一种可部署格式中，从而支持在 CPU 设备上高效执行程序。

要编译程序以便在 CPU 上运行：

1. 选择 LLVM 支持的 CPU 目标。默认情况下，IREE 包含以下 LLVM targets：

    * X86
    * ARM
    * AArch64
    * RISCV

    其他目标也可能可用，但项目内的测试覆盖和性能优化主要集中在上述列表中的目标。

2. 选择 IREE 支持的一种可执行文件格式：

    | 可执行文件格式     | 说明                                                   |
    | ----------------- | ----------------------------------------------------- |
    | Embedded ELF      |（默认）可移植、高性能的动态库                         |
    | System library    | 平台专用的动态库（.so、.dll 等）                      |
    | VMVX              | 参考目标                                               |

运行时可使用 IREE 的 CPU HAL（硬件抽象层）设备加载 CPU 可执行文件：

* `local-task`：基于 IREE 的“task”系统构建的异步、多线程设备
* `local-sync`：同步、单线程设备，会内联执行工作

## :octicons-download-16: 前置条件

### 获取 IREE 编译器

#### :octicons-download-16: 从 release 下载编译器

Python 软件包通过多个渠道分发。更多详情请参阅
[Python Bindings](../../reference/bindings/python.md) 页面。核心的
[`iree-base-compiler`](https://pypi.org/project/iree-base-compiler/)
软件包包含编译器工具：

--8<-- "docs/website/docs/guides/deployment-configurations/snippets/_iree-compiler-from-release.md"

#### :material-hammer-wrench: 从源代码构建编译器

请先按照[入门指南](../../building-from-source/getting-started.md)中的说明，为宿主平台构建 IREE。所有平台默认都会编译 `llvm-cpu` 编译器后端；不过，配置时仍应确保 CMake 选项 `IREE_TARGET_BACKEND_LLVM_CPU` 为 `ON`。

!!! tip
    `iree-compile` 会构建在 `iree-build/tools/` 目录下。你可能需要将此路径添加到系统的 `PATH` 环境变量中。

### 获取 IREE 运行时

你需要获取支持 local CPU HAL 驱动程序的 IREE 运行时，以及应用所需的相应可执行文件加载器。

#### :octicons-download-16: 从 release 下载运行时

Python 软件包通过多个渠道分发。更多详情请参阅
[Python Bindings](../../reference/bindings/python.md) 页面。核心的
[`iree-base-runtime`](https://pypi.org/project/iree-base-runtime/)
软件包包含 local CPU HAL 驱动程序：

--8<-- "docs/website/docs/guides/deployment-configurations/snippets/_iree-runtime-from-release.md"

#### :material-hammer-wrench: 从源代码构建运行时

请先按照[从源代码构建](../../building-from-source/index.md)页面之一中的说明，为目标平台构建 IREE。所有平台默认都会编译 local CPU HAL 驱动程序和设备；不过，配置时仍应确保 CMake 选项 `IREE_HAL_DRIVER_LOCAL_TASK` 和 `IREE_HAL_EXECUTABLE_LOADER_EMBEDDED_ELF`（或其他可执行文件加载器选项）为 `ON`。

#### :octicons-checklist-24: 检查 CPU 设备

你可以通过查看 `local-sync` 和 `local-task` drivers 与 devices 来检查 CPU 支持情况：

```console hl_lines="10-11"
--8<-- "docs/website/docs/guides/deployment-configurations/snippets/_iree-run-module-driver-list.md:2"
```

```console hl_lines="4-5"
--8<-- "docs/website/docs/guides/deployment-configurations/snippets/_iree-run-module-device-list-amd.md"
```

## 编译并运行程序

准备工作完成后，我们就可以编译模型并运行了。

### :octicons-file-code-16: 编译程序

--8<-- "docs/website/docs/guides/deployment-configurations/snippets/_iree-import-onnx-mobilenet.md"

然后运行以下命令，使用 `local` device、`llvm-cpu` 目标编译后端以及推荐的优化标志进行编译：

``` shell hl_lines="2-4"
iree-compile \
    --iree-hal-target-device=local \
    --iree-hal-local-target-device-backends=llvm-cpu \
    --iree-llvmcpu-target-cpu=host \
    --iree-opt-level=O2 \
    --iree-opt-data-tiling \
    mobilenetv2.mlir -o mobilenet_cpu.vmfb
```

???+ tip "提示 - 目标 CPU 和 CPU 特性"

    默认情况下，编译器会使用通用 CPU 目标，这会导致性能较差。应使用以下选项之一选择目标 CPU 或目标 CPU 特性集：

    * `--iree-llvmcpu-target-cpu=...`
    * `--iree-llvmcpu-target-cpu-features=...`

    如果不进行交叉编译，在大多数设备上通常只需传入 `--iree-llvmcpu-target-cpu=host`。

--8<-- "docs/website/docs/guides/deployment-configurations/snippets/_iree-optimization-options.md"

???+ tip "提示 - Data-Tiling（数据分块）"

    使用 `--iree-opt-data-tiling` 启用 IREE 开发人员一直在推进的优化。出于多种原因，此选项默认关闭，但面向 CPU 时，用户曾使用它来提升性能。更多详情请参阅
    [Data-Tiling](../../reference/optimization-options.md#data-tiling-iree-opt-data-tiling-off)。

#### 选择 CPU 目标

`--iree-llvmcpu-target-triple` 标志告知编译器为特定类型的 CPU 生成代码。你可以通过 `iree-compile --iree-llvmcpu-list-targets` 查看支持的目标列表；也可以使用默认值 `"host"`，让 LLVM 根据宿主机器推断 triple（例如 `x86_64-linux-gnu`）。

```console
$ iree-compile --iree-llvmcpu-list-targets

  Registered Targets:
    aarch64    - AArch64 (little endian)
    aarch64_32 - AArch64 (little endian ILP32)
    aarch64_be - AArch64 (big endian)
    arm        - ARM
    arm64      - ARM64 (little endian)
    arm64_32   - ARM64 (little endian ILP32)
    armeb      - ARM (big endian)
    riscv32    - 32-bit RISC-V
    riscv64    - 64-bit RISC-V
    wasm32     - WebAssembly 32-bit
    wasm64     - WebAssembly 64-bit
    x86        - 32-bit X86: Pentium-Pro and above
    x86-64     - 64-bit X86: EM64T and AMD64
```

### :octicons-terminal-16: 运行已编译的程序

运行编译后的程序：

``` shell hl_lines="2"
iree-run-module \
    --device=local-task \
    --module=mobilenet_cpu.vmfb \
    --function=torch-jit-export \
    --input="1x3x224x224xf32=0"
```

以上命令假设模型中导出的函数名为 `torch-jit-export`，并且该函数需要一张 224x224 RGB 图像。为简洁起见，这里输入的是一张像素值全为 0 的图像；有关如何指定具体值的格式，请参阅 `iree-run-module --help`。

<!-- TODO(??): measuring performance -->

<!-- TODO(??): troubleshooting -->
