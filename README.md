# btop-xtu

A community fork of [btop++](https://github.com/aristocratos/btop) that focuses on
GPU support beyond the three vendors upstream knows about — in particular the
Chinese GPU vendors (Moore Threads / MUSA first), plus a few usability and
layout improvements.

[btop++](https://github.com/aristocratos/btop) 的社区分支，专注于补齐上游缺失的 GPU 支持，
尤其是摩尔线程（MUSA）等国产 GPU，并附带一些界面与布局上的改进。

![Linux](https://img.shields.io/badge/-Linux-grey?logo=linux)
![macOS](https://img.shields.io/badge/-OSX-black?logo=apple)
![FreeBSD](https://img.shields.io/badge/-FreeBSD-red?logo=freebsd)
![NetBSD](https://img.shields.io/badge/-NetBSD-black?logo=netbsd)
![OpenBSD](https://img.shields.io/badge/-OpenBSD-black?logo=openbsd)
![MUSA](https://img.shields.io/badge/GPU-MUSA-1f6feb)

---

## Why this fork / 为什么有这个分支

Upstream btop++ can only read GPU counters through NVML (NVIDIA), ROCm SMI (AMD),
Intel's i915 PMU and Apple's IOReport. Anything else — Moore Threads MUSA,
Cambricon MLU, Huawei Ascend, Hygon DCU, MetaX, Iluvatar, Biren — shows up as an
ordinary PCI device, so the GPU box simply never appears. Patches for those
vendors sit in the queue for months, so this fork maintains them out of tree.

Upstream 的 GPU 监控只接了 NVML（NVIDIA）、ROCm SMI（AMD）、Intel i915 PMU 和 Apple
IOReport 四条路。摩尔线程 MUSA、寒武纪 MLU、华为昇腾、海光 DCU、沐曦、天数智芯、壁仞
这些卡在 btop++ 里都只是普通 PCI 设备，GPU 面板根本不会出现。相关补丁在上游长期排队，
所以这个分支把这类支持独立维护出来。

## Features / 特性

### MUSA (Moore Threads) GPU support / 摩尔线程 MUSA 支持

Reads utilization, VRAM, temperature, power, clocks, encoder/decoder usage and
PCIe throughput for MTT S-series cards through the MTML library
(`libmtml.so`, the vendor's NVML equivalent). The library is `dlopen`ed at
runtime, so the binary still runs on machines without a Moore Threads GPU and
needs no MUSA SDK at build time.

通过 MTML 库（`libmtml.so`，摩尔线程对标 NVML 的接口）读取 MTT S 系列的
GPU 利用率、显存、温度、功耗、频率、编解码占用和 PCIe 吞吐。库是运行时 `dlopen`
的，所以没有摩尔线程显卡的机器上照常运行，编译时也不需要 MUSA SDK。

### Side-by-side GPU boxes / GPU 面板横向自适应

The GPU boxes used to be stacked full-width, wasting most of a wide terminal.
They are now laid out on a grid: `gpu_box_columns = "Auto"` fits as many columns
as the terminal allows, a box drops from Full to Compact to Minimal detail
instead of overflowing, and `1` restores the old stacked layout exactly.

原来的 GPU 面板整宽纵向堆叠，宽终端下大半屏都是空白。现在按网格排布：
`gpu_box_columns = "Auto"` 会根据终端宽度自动决定列数，空间不够时面板自动从
Full 降到 Compact、Minimal 而不是画崩；设成 `1` 则完全恢复旧的堆叠布局。

### Own identity / 独立身份

`btop-xtu` installs its own binary, config directory (`~/.config/btop-xtu`),
state log (`~/.local/state/btop-xtu.log`) and theme directory
(`share/btop-xtu`), so it can live next to an installed `btop` without either
one clobbering the other's settings.

`btop-xtu` 有自己的可执行文件名、配置目录（`~/.config/btop-xtu`）、日志
（`~/.local/state/btop-xtu.log`）和主题目录（`share/btop-xtu`），可以和系统里
已有的 `btop` 并存，互不影响配置。

## GPU support matrix / GPU 支持矩阵

| Vendor / 厂商 | Backend / 后端 | Status / 状态 |
|---|---|---|
| NVIDIA | NVML | upstream / 上游已有 |
| AMD | ROCm SMI | upstream / 上游已有 |
| Intel | i915 PMU | upstream / 上游已有 |
| Apple | IOReport | upstream / 上游已有 |
| Moore Threads 摩尔线程 | MTML (`mtmlx*`) | **this fork / 本分支** |
| Cambricon 寒武纪 | cnmon / CNNL | planned / 计划中 |
| Huawei Ascend 昇腾 | `npu-smi` / CANN | planned / 计划中 |
| Hygon DCU 海光 | ROCm SMI | planned / 计划中 |
| MetaX 沐曦 / Iluvatar 天数 / Biren 壁仞 | vendor SMI | planned / 计划中 |

"Planned" means the backend is not written yet — issues and sample output from
real hardware are very welcome.

“计划中”表示后端尚未实现，欢迎提供真机上的 `*-smi` 输出样例或直接提 PR。

## Build / 构建

Requirements: a C++20 compiler (GCC 11+ or Clang 14+, GCC 14 recommended),
CMake 3.25+, and the usual build tools.

依赖：C++20 编译器（GCC 11+ / Clang 14+，建议 GCC 14）、CMake 3.25+ 和常规构建工具。

```bash
git clone https://github.com/yeungtuzi/btop-xtu.git
cd btop-xtu

cmake -B build -DCMAKE_BUILD_TYPE=Release \
      -DBTOP_GPU=ON -DBTOP_MUSA=ON
cmake --build build -j"$(nproc)"
sudo cmake --install build          # -> /usr/local/bin/btop-xtu
```

Useful options / 常用选项:

| Option | Default | Meaning / 说明 |
|---|---|---|
| `BTOP_GPU` | `ON` | build any GPU support / 是否编译 GPU 支持 |
| `BTOP_MUSA` | `ON` | Moore Threads MTML backend / 摩尔线程后端 |
| `BTOP_LTO` | `ON` (Release) | link-time optimisation / LTO |
| `BTOP_STATIC` | `OFF` | fully static binary / 全静态链接 |

On a Moore Threads machine, the runtime needs the driver and
`libmthreads-mtml` (`libmtml.so.1`); the headers come from
`libmthreads-mtml-dev` if you want to build against the API directly.

在摩尔线程机器上，运行时需要驱动和 `libmthreads-mtml`（`libmtml.so.1`）；
如果想直接对着头文件开发，再装 `libmthreads-mtml-dev`。

## Usage / 使用

```bash
btop-xtu                 # start
btop-xtu -c my.conf      # alternate config
btop-xtu --help
```

On the very first run `btop-xtu` gives every GPU it recognises its own box, so a
GPU machine shows the panels without any config editing. Cards added later are
not picked up automatically — add the corresponding `gpuN` to `shown_boxes` by
hand (or delete the config file to start over).

首次运行时，`btop-xtu` 会自动为每张识别到的 GPU 建一个面板，GPU 机器开箱即用，
不用手改配置。之后新插的卡不会自动出现——手动往 `shown_boxes` 里加对应的 `gpuN`
即可（或者删掉配置文件重新生成）。

```ini
shown_boxes = "cpu mem net proc gpu0 gpu1"
gpu_box_columns = "Auto"     # or 1..6 / 或 1~6 强制列数
shown_gpus = "nvidia amd intel apple musa"
```

## How the MUSA backend works / MUSA 后端实现

MTML ships two APIs: the object-oriented `mtml*` one and `mtmlx*`, a drop-in clone
of NVML (same function names, same struct layouts, same enums). The backend
`dlopen`s `libmtml.so.1`, resolves the `mtmlx*` entry points and reuses the data
flow of btop's NVIDIA backend: `mtmlxInit_v2` → `mtmlxDeviceGetCount_v2` →
`mtmlxDeviceGetHandleByIndex_v2`, then per-collect calls for
`GetUtilizationRates`, `GetMemoryInfo`, `GetTemperature`, `GetPowerUsage`,
`GetClockInfo`, `GetEncoder/DecoderUtilization` and `GetPcieThroughput`.
Missing calls are detected at init and the corresponding field is disabled, so a
driver that only implements part of the API degrades gracefully.

MTML 提供两套接口：面向对象的 `mtml*`，以及与 NVML 完全同构的 `mtmlx*`（同名函数、
同结构体布局、同枚举）。后端 `dlopen` `libmtml.so.1` 后解析 `mtmlx*` 符号，
复用 btop 原有 NVIDIA 后端的数据流：`mtmlxInit_v2` → `mtmlxDeviceGetCount_v2` →
`mtmlxDeviceGetHandleByIndex_v2`，再按需调用 `GetUtilizationRates`、`GetMemoryInfo`、
`GetTemperature`、`GetPowerUsage`、`GetClockInfo`、`GetEncoder/DecoderUtilization`、
`GetPcieThroughput`。初始化时会探测每个接口，缺失的字段自动隐藏，驱动只实现一部分
API 也能正常显示。

Known limitation: the mtgpu driver does not expose a board power limit
(`mthreads-gmi` prints `N/A` as well), so the power bar assumes 450 W — the MTT
S4000 board power. Change the constant in `Gpu::Musa::init()` for other cards.

已知限制：mtgpu 驱动不暴露板卡功耗上限（`mthreads-gmi` 同样显示 `N/A`），
功耗条按 450 W（MTT S4000 板卡功耗）估算。换其他型号时改
`Gpu::Musa::init()` 里的常量即可。

## Roadmap / 路线图

* More domestic GPU backends: Cambricon, Ascend, Hygon DCU, MetaX, Iluvatar.
* 更多国产 GPU 后端：寒武纪、昇腾、海光 DCU、沐曦、天数智芯。
* Per-GPU process list inside the GPU box (upstream has a TODO for this).
* GPU 面板内展示占用该卡的进程列表。
* Keep the fork rebased on upstream releases so fixes flow both ways.
* 持续跟进上游版本，让修复可以双向流动。

## Relationship with upstream / 与上游的关系

Based on upstream v1.4.7. The GPU grid layout comes from the `feat/gpu-box-columns`
work (upstream PR pending, no response so far); the MUSA backend is fork-only.
Both are kept as separate commits so they can be re-offered upstream later.

基于上游 v1.4.7。GPU 网格布局来自 `feat/gpu-box-columns` 的工作（上游 PR 至今没有回应），
MUSA 后端为本分支独有。两者都保持为独立提交，方便日后重新向上游提交。

## License / 许可证

Apache-2.0, same as upstream btop++.

与上游 btop++ 一致，采用 Apache-2.0。

## Upstream documentation / 上游文档

For everything not covered here (themes, keybindings, the full option list), see
the [upstream README](https://github.com/aristocratos/btop#readme).

主题、快捷键、完整配置项等未在此覆盖的内容，请参考
[上游 README](https://github.com/aristocratos/btop#readme)。
