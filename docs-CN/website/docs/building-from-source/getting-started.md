---
icon: octicons/bookmark-16
---

# 快速入门

## :octicons-download-16: 前置条件

IREE 可以使用 [CMake](https://cmake.org/) 从源代码构建。我们还推荐使用
[Ninja](https://ninja-build.org/) 作为 CMake generator，以及
[clang](https://clang.llvm.org/) 或 MSVC C/C++ 编译器。

??? Note "注意 - 其他 CMake generator 和编译器"
    IREE 开发者和 CI 主要使用 Ninja、clang 和 MSVC。其他配置（包括 Makefile
    generator 和 gcc）均按“尽力支持”处理。欢迎提交补丁以改进支持。

=== ":fontawesome-brands-linux: Linux"

    1. 安装编译器/链接器（通常安装 "clang" 和 "lld" 软件包）

    2. 安装 [CMake](https://cmake.org/download/)（通常安装 "cmake" 软件包）

    3. 安装 [Ninja](https://ninja-build.org/)（通常安装 "ninja-build"
       软件包）

    在 Debian/Ubuntu 上：

    ``` shell
    sudo apt install cmake ninja-build clang lld
    ```

=== ":fontawesome-brands-apple: macOS"

    1. 安装 [CMake](https://cmake.org/download/)

    2. 安装 [Ninja](https://ninja-build.org/)

    如果使用 Homebrew：

    ``` shell
    brew install cmake ninja
    ```

=== ":fontawesome-brands-windows: Windows"

    1. 从 Visual Studio 或 "Tools for Visual Studio" 安装 MSVC，下载地址见
       [官方页面](https://visualstudio.microsoft.com/downloads/)

    2. 从 [官方页面](https://cmake.org/download/) 安装 CMake

    3. 从 [官方网站](https://ninja-build.org/) 安装 Ninja

    !!! note
        若要在命令行上构建，请运行 `vcvarsall.bat` 初始化 MSVC。
        详情请参阅
        [官方文档](https://docs.microsoft.com/en-us/cpp/build/building-on-the-command-line)。

<!-- TODO(#12921): add notes about Docker and/or dev containers  -->

## :octicons-rocket-16: 快速开始：克隆并构建

使用 [Git](https://git-scm.com/) 克隆 IREE 仓库并初始化其
submodule：

``` shell
git clone https://github.com/iree-org/iree.git
cd iree
git submodule update --init
```

最基本的 CMake 工作流程如下：

``` shell
# Configure
cmake -G Ninja -B ../iree-build/ .

# Build
cmake --build ../iree-build/
```

!!! Caution "注意 - 构建速度较慢"
    编译器的构建过程较复杂。你需要性能强劲的机器，并根据下一节调整
    设置。2023 年，我们观察到在 64 核 Linux 机器上构建需要约 5-10 分钟。

    如果用例允许，使用 `-DIREE_BUILD_COMPILER=OFF` 禁用编译器构建可以大幅简化
    build。

## :octicons-sliders-16: 配置设置

配置步骤应根据你的 build 环境进行定制。这些设置可以显著缩短编译和链接时间。

<!-- TODO(#5804): add notes about CMake presets?  -->

=== ":fontawesome-brands-linux: Linux"

    ``` shell
    # Recommended development options using clang and lld:
    cmake -G Ninja -B ../iree-build/ -S . \
        -DCMAKE_BUILD_TYPE=RelWithDebInfo \
        -DIREE_ENABLE_ASSERTIONS=ON \
        -DIREE_ENABLE_SPLIT_DWARF=ON \
        -DIREE_ENABLE_THIN_ARCHIVES=ON \
        -DCMAKE_C_COMPILER=clang \
        -DCMAKE_CXX_COMPILER=clang++ \
        -DIREE_ENABLE_LLD=ON
    ```

=== ":fontawesome-brands-apple: macOS"

    ``` shell
    # Recommended development options using clang and lld:
    cmake -G Ninja -B ../iree-build/ -S . \
        -DCMAKE_BUILD_TYPE=RelWithDebInfo \
        -DIREE_ENABLE_ASSERTIONS=ON \
        -DIREE_ENABLE_SPLIT_DWARF=ON \
        -DCMAKE_C_COMPILER=clang \
        -DCMAKE_CXX_COMPILER=clang++ \
        -DIREE_ENABLE_LLD=ON
    ```

    如果 `CMAKE_AR` 变量已定义，且指向 GNU binutils 或 LLVM `ar` 程序的路径
    （从而覆盖默认的 Apple `ar`），也可以添加 `-DIREE_ENABLE_THIN_ARCHIVES=ON`。

=== ":fontawesome-brands-windows: Windows"

    ``` shell
    # Recommended development options:
    cmake -G Ninja -B ../iree-build/ -S . \
        -DCMAKE_BUILD_TYPE=RelWithDebInfo \
        -DIREE_ENABLE_ASSERTIONS=ON
    ```

!!! Note "注意 - thin archive 与动态编译器插件"
    `-DIREE_ENABLE_THIN_ARCHIVES=ON` 不能与
    `-DIREE_EXPERIMENTAL_COMPILER_DYNAMIC_PLUGINS=ON` 同时使用；同时配置这两项会失败。动态
    plugins 会链接到一份编译器符号重命名后的副本，而重命名无法改写 thin archive。

???+ Tip "提示 - CMAKE_BUILD_TYPE 的取值"
    我们建议默认使用 `RelWithDebInfo` build type，以兼顾调试信息和性能。
    `Debug`、`Release` 和 `MinSizeRel` build type 适用于更具体的场景。请注意，
    多项有用的 LLVM 调试功能仅在 `Debug` build 中可用。一般详情请参阅
    [CMake 官方文档](https://cmake.org/cmake/help/latest/variable/CMAKE_BUILD_TYPE.html)。

???+ Tip "提示 - 使用 ccache 加快重新编译"
    我们建议在 CMake 中使用 [`ccache`](https://ccache.dev/)，尤其是在重新构建
    编译器时。要使用它，请通过以下选项配置 CMake：

    ``` shell
    -DCMAKE_C_COMPILER_LAUNCHER=ccache -DCMAKE_CXX_COMPILER_LAUNCHER=ccache
    ```

    另请参阅我们的 [ccache 开发文档](../developers/building/cmake-with-ccache.md)。

### :octicons-gear-16: 可选组件

启用的组件和其他配置可以通过
[CMake 选项](../developers/building/cmake-options.md)更改，选项列于根目录的
[`CMakeLists.txt`](https://github.com/iree-org/iree/blob/main/CMakeLists.txt)。
我们还维护了一些适用于常见配置的
[CMake presets](https://cmake.org/cmake/help/latest/manual/cmake-presets.7.html)，位于
[`build_tools/cmake/presets`](https://github.com/iree-org/iree/tree/main/build_tools/cmake/presets)。

默认情况下，CMake build 包含：

* 所有小型到中型的[编译器 target](../developers/building/cmake-options.md#iree_target_backend_)
  （`llvm-cpu`、`vulkan-spirv` 等）
* 所有[运行时 HAL driver](../developers/building/cmake-options.md#iree_hal_driver_)
  （`local-task`、`cuda`、`vulkan` 等）
* 所有[编译器输入格式](../developers/building/cmake-options.md#iree_input_)
  （PyTorch、StableHLO、TOSA 等）
* 所有编译器输出格式（VM 字节码、C）

默认 build _不_ 包含：

* CUDA 和 ROCm/HIP targets
* 编译器或运行时的 Python 及其他语言绑定
* AddressSanitizer 或 tracing instrumentation 等高级功能
* 实验性组件

!!! example "配置示例"

    === "禁用除 CPU 外的所有 backends"

        此配置命令将会：

        * 禁用所有编译器 target backends，然后仅启用 `llvm-cpu`
        * 禁用所有运行时 HAL drivers，然后仅启用 CPU“local”运行时
          HAL drivers

        ``` shell
        cmake -G Ninja -B ../iree-build/ -S . \
            -DIREE_TARGET_BACKEND_DEFAULTS=OFF \
            -DIREE_TARGET_BACKEND_LLVM_CPU=ON \
            -DIREE_HAL_DRIVER_DEFAULTS=OFF \
            -DIREE_HAL_DRIVER_LOCAL_SYNC=ON \
            -DIREE_HAL_DRIVER_LOCAL_TASK=ON
        ```

    === "启用 CUDA"

        此配置命令将会：

        * 启用 CUDA 编译器 target backend
        * 启用 CUDA 运行时 HAL driver

        ``` shell
        cmake -G Ninja -B ../iree-build/ -S . \
            -DIREE_TARGET_BACKEND_CUDA=ON \
            -DIREE_HAL_DRIVER_CUDA=ON
        ```

    === "启用 ROCm/HIP"

        此配置命令将会：

        * 启用 ROCm 编译器 target backend
        * 启用 HIP 运行时 HAL driver

        ``` shell
        cmake -G Ninja -B ../iree-build/ -S . \
            -DIREE_TARGET_BACKEND_ROCM=ON \
            -DIREE_HAL_DRIVER_HIP=ON
        ```

### 扩展和集成

在其他项目中使用 IREE 时，你可以注册编译器插件和运行时 HAL drivers。
你也可以使用自己的 LLVM 副本及其他工具。详情请参阅根目录的
[`CMakeLists.txt`](https://github.com/iree-org/iree/blob/main/CMakeLists.txt)。

## :octicons-code-16: 测试和示例

### 运行测试

测试通过 [ctest](https://cmake.org/cmake/help/latest/manual/ctest.1.html) 运行。
若要构建并运行项目核心测试：

``` shell
# Build default targets
cmake --build ../iree-build/

# Run tests
ctest --test-dir ../iree-build/
```

!!! Caution
    此方式有两项限制：

    1. 默认 build 不包含大型测试
    2. 有些测试需要 GPU 等硬件，在不支持的系统上会失败

若要构建并运行所有测试：

``` shell
# 1. Build default targets
cmake --build ../iree-build/

# 2. Build test dependencies
cmake --build ../iree-build/ --target iree-test-deps

# 3. Run tests
ctest --test-dir ../iree-build/


# Or combine all steps using a utility target
cmake --build ../iree-build --target iree-run-tests
```

若只运行特定测试，我们提供了一个
[辅助脚本](https://github.com/iree-org/iree/blob/main/build_tools/cmake/ctest_all.sh)，
它会将环境变量转换为 ctest filters：

``` shell
# Run default tests
./build_tools/cmake/ctest_all.sh ../iree-build

# Run tests, turning CUDA on and Vulkan off
export IREE_CUDA_DISABLE=0
export IREE_VULKAN_DISABLE=1
./build_tools/cmake/ctest_all.sh ../iree-build
```

### 运行示例

``` shell
# Build
cmake --build ../iree-build/

# Run a standalone sample application
../iree-build/runtime/src/iree/runtime/demo/hello_world_embedded
# 4xf32=1 1.1 1.2 1.3
#  *
# 4xf32=10 100 1000 10000
#  =
# 4xf32=10 110 1200 13000

# Try out the developer tools
ls ../iree-build/tools/
../iree-build/tools/iree-compile --help
../iree-build/tools/iree-run-module --help
```

## :simple-python: Python 绑定

Python 软件包可以从源代码构建，也可以从我们的发布版本安装。
有关绑定本身的详情，请参阅 [Python 绑定](../reference/bindings/python.md)页面。

### 依赖项

你需要 Python >=3.10。IREE 支持
[尚未 EOL 的 Python 版本](https://endoflife.date/python)，遵循与 LLVM 最低所需 Python
版本相同的生命周期策略——参见
[LLVM 的 Python 支持策略](https://discourse.llvm.org/t/rfc-adopt-regularly-scheduled-python-minimum-version-bumps/88841)。

???+ Tip "提示 - 管理 Python 版本"
    请确认 'python' 指向你预期使用的版本：

    === ":fontawesome-brands-linux: Linux"

        请注意，在多 Python 版本系统上，命令可能带有版本后缀；在许多可以共存
        python2 和 python3 的 Linux 系统上，你可能还需要使用 `python3`。

        ``` shell
        which python
        python --version
        ```

    === ":fontawesome-brands-apple: macOS"

        请注意，在多 Python 版本系统上，命令可能带有版本后缀；在可以共存
        python2 和 python3 的 macOS 上，你可能还需要使用 `python3`。

        ``` shell
        which python
        python --version
        ```

    === ":fontawesome-brands-windows: Windows"
        [Windows 的 Python launcher](https://docs.python.org/3/using/windows.html#python-launcher-for-windows) (`py`)
        可以帮助管理不同版本。

        ``` powershell
        which python
        python --version
        py --list-paths
        ```

???+ Tip "提示 - 虚拟环境"
    我们建议使用虚拟环境（例如通过 `venv`）管理 Python 软件包
    （[介绍](https://docs.python.org/3/library/venv.html)，
    [教程](https://docs.python.org/3/tutorial/venv.html)）：

    === ":fontawesome-brands-linux: Linux"

        ``` shell
        python -m venv .venv
        source .venv/bin/activate
        ```

    === ":fontawesome-brands-apple: macOS"

        ``` shell
        python -m venv .venv
        source .venv/bin/activate
        ```

    === ":fontawesome-brands-windows: Windows"

        ``` powershell
        python -m venv .venv
        .venv\Scripts\activate.bat
        ```

    完成后，运行 `deactivate`。

``` shell
# Upgrade PIP before installing other requirements
python -m pip install --upgrade pip

# Install IREE build requirements
python -m pip install -r runtime/bindings/python/iree/runtime/build_requirements.txt
```

### 使用 CMake 构建

若要构建 Python 绑定，请使用 `IREE_BUILD_PYTHON_BINDINGS` 选项配置 CMake。
我们还建议显式设置要使用的 Python 可执行文件，即 `Python3_EXECUTABLE`：

``` shell
# Configure (including other options as discussed above)
cmake -G Ninja -B ../iree-build/ \
  -DIREE_BUILD_PYTHON_BINDINGS=ON  \
  -DPython3_EXECUTABLE="$(which python3)" \
  .

# Build
cmake --build ../iree-build/
```

### 使用 Python 绑定

安装 Python 绑定有两种方法：创建 editable wheel，或扩展 `PYTHONPATH`。

#### 选项 A：将绑定安装为 editable wheels

此方法会将 build tree 中的文件作为 editable wheel 链接到你的 Python 软件包目录。

=== ":fontawesome-brands-linux: Linux"

    ``` shell
    CMAKE_INSTALL_METHOD=ABS_SYMLINK python -m pip install -e ../iree-build/compiler
    CMAKE_INSTALL_METHOD=ABS_SYMLINK python -m pip install -e ../iree-build/runtime
    ```

=== ":fontawesome-brands-apple: macOS"

    ``` shell
    CMAKE_INSTALL_METHOD=ABS_SYMLINK python -m pip install -e ../iree-build/compiler
    CMAKE_INSTALL_METHOD=ABS_SYMLINK python -m pip install -e ../iree-build/runtime
    ```

=== ":fontawesome-brands-windows: Windows"

    ``` powershell
    $env:CMAKE_INSTALL_MODE="ABS_SYMLINK"
    python -m pip install -e ..\iree-build\compiler
    python -m pip install -e ..\iree-build\runtime
    $env:CMAKE_INSTALL_MODE=null
    ```

#### 选项 B：扩展 PYTHONPATH

此方法能更有效地反映 build 目录的状态，但如果忘记 source 环境变量，
就容易出错。

将 IREE 的 `bindings/python` 路径添加到 `PYTHONPATH`，并尝试导入：

=== ":fontawesome-brands-linux: Linux"

    ``` shell
    source ../iree-build/.env && export PYTHONPATH
    # The 'PYTHONPATH' environment variable should now contain
    #   iree-build/compiler/bindings/python;iree-build/runtime/bindings/python

    python -c "import iree.compiler; help(iree.compiler)"
    python -c "import iree.runtime; help(iree.runtime)"
    ```

=== ":fontawesome-brands-apple: macOS"

    ``` shell
    source ../iree-build/.env && export PYTHONPATH
    # The 'PYTHONPATH' environment variable should now contain
    #   iree-build/compiler/bindings/python;iree-build/runtime/bindings/python

    python -c "import iree.compiler; help(iree.compiler)"
    python -c "import iree.runtime; help(iree.runtime)"
    ```

=== ":fontawesome-brands-windows: Windows"

    ``` powershell
    ..\iree-build\.env.ps1  # or ..\iree-build\.env.bat
    # The 'PYTHONPATH' environment variable should now contain
    #   iree-build/compiler/bindings/python;iree-build/runtime/bindings/python

    python -c "import iree.compiler; help(iree.compiler)"
    python -c "import iree.runtime; help(iree.runtime)"
    ```

#### Tensorflow/TFLite 绑定

使用 IREE 的 TensorFlow/TFLite 导入器还需要几个额外步骤：

``` shell
# Install test requirements
python -m pip install -r integrations/tensorflow/test/requirements.txt

# Install pure Python packages (no build required)
# You may use `pip install -e` here to create an editable wheel.
python -m pip install integrations/tensorflow/python_projects/iree_tf
python -m pip install integrations/tensorflow/python_projects/iree_tflite

# Then test the tools:
iree-import-tf --help
iree-import-tflite --help
```
