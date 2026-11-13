# Dmitry Galoshin (@BannedForWhat)

**Linux Platform & AI Infrastructure Engineer | Systems DevSecOps**  
Focused on low-level Linux systems programming (C++17/C++20 bare-metal), deterministic zero-allocation memory models, kernel DRM/TTM telemetry, unprivileged containerless runtime sandboxing, and resilient L7 AI gateways.

---

### Architecture: Sentinel-Mesh & Agent-Jail Infrastructure

```text
[ Hardware: AMD RDNA 4 (Navi 48 XTX, gfx1201) / Linux Kernel 6.x / Arch Linux ]
                                │
                                ▼ persistent FDs, pread(fd, buf, 64, 0)
┌─────────────────────────────────────────────────────────────────────────────┐
│ 1. amdgpu-sentinel (L1/L2 Kernel Telemetry Daemon)                          │
│    • Bare-metal C++17 (-fno-exceptions, -fno-rtti, -O2)                     │
│    • Reactor: epoll_pwait2 + timerfd (CLOCK_MONOTONIC) + signalfd4          │
│    • Footprint: 420 KB RSS, 0 heap allocations in hot loop                  │
└──────────────────────────────────────┬──────────────────────────────────────┘
                                       ▼ POSIX SHM (mmap, Seqlock <50 ns)
                               /dev/shm/vram_guard
                                       │
                                       ▼ Zero-copy atomic read
┌─────────────────────────────────────────────────────────────────────────────┐
│ 2. prompt-mesh (L7 VRAM-Aware AI Gateway)                                   │
│    • Pure Python 3.12 asyncio (Port :8000, zero third-party dependencies)   │
│    • Anti-OOM: Middle-Out Pruning triggered by VRAM >85% or ttm_alert == 1  │
│    • Active Slot Cancellation: immediate TCP reset + POST /slots/erase      │
│    • Recency Bias Anchors & Intent Router (:8080 Code, :8082 Architecture)  │
└──────────────────────────────────────┬──────────────────────────────────────┘
                                       │
            ┌──────────────────────────┴──────────────────────────┐
            ▼ HTTP Stream                                         ▼ Pathname UNIX Socket
┌───────────────────────────────┐             ┌───────────────────────────────────────────────┐
│ llama-server (:8080 / :8082)  │             │ 3. agent-jail (Runtime Security Sandbox)      │
│ • ROCm 7.x / Mesa RADV Vulkan │             │    • C++20 containerless isolator (<1 ms cold)│
│ • Qwen2.5-Coder-14B (Q5_K_M)  │             │    • 6 Namespaces (user, mount, pid, net, ...)│
│ • 100% VRAM, 32K context q8_0 │             │    • Arch Linux Merged /usr VFS + pivot_root  │
└───────────────────────────────┘             │    • Landlock LSM (ABI v5) + Seccomp-BPF      │
                                              │    • cgroups v2: memory.swap.max=0, pids.max  │
                                              │    • Non-blocking PTY master/slave forwarder  │
                                              └───────────────────────────────────────────────┘

───────────────────────────────────────────────────────────────────────────────────────────────
4. sentinel-mesh-ops (DevSecOps & Reliability Verification Harness)
   • Headless Bubblewrap (bwrap) sysfs mocking in GitHub Actions (Ubuntu 24.04 under AppArmor)
   • Linker-level proof of zero allocations via GNU ld: -Wl,--wrap=malloc/calloc/realloc/free
   • Isolated multi-compiler sanitizer matrix: ASan / UBSan / TSan (Seqlock race verification)
```

---

### Verified Production Repositories

* [**agent-jail**](https://github.com/BannedForWhat/agent-jail) — Sub-millisecond (<1 ms) unprivileged containerless runtime sandbox for autonomous AI coding agents in C++20. Implements 6 Linux namespaces, Arch Merged `/usr` VFS with `pivot_root(2)`, Landlock LSM (ABI v5), handcrafted Seccomp-BPF filters (`clone3 -> ENOSYS`, `AF_UNIX` enforcement), direct cgroups v2 resource capping (`memory.swap.max=0`), and asynchronous PTY multiplexing.
* [**amdgpu-sentinel**](https://github.com/BannedForWhat/amdgpu-sentinel) — Zero-allocation C++17 bare-metal telemetry daemon for AMD DRM/TTM sysfs. Uses lockless `pread()` at offset 0 to bypass VFS `f_pos_lock`, single-threaded `epoll_pwait2` reactor with monotonic `timerfd`, and atomic 64-byte Seqlock state export to `/dev/shm/vram_guard` (420 KB RSS, systemd user service sandbox).
* [**prompt-mesh**](https://github.com/BannedForWhat/prompt-mesh) — Hardware-aware, high-performance L7 AI Gateway on pure Python 3.12 `asyncio`. Reads kernel telemetry via zero-copy `mmap` struct unpacking (<100 ns), executes microsecond Middle-Out Pruning on VRAM pressure, injects Recency Bias Anchors, and terminates backend decoder loops via active socket resets on client disconnect.
* [**sentinel-mesh-ops**](https://github.com/BannedForWhat/sentinel-mesh-ops) — Comprehensive CI/CD reliability harness. Features headless sysfs tree mocking via rootless Bubblewrap with dynamic AppArmor profiles, binary zero-allocation proof via GNU ld symbol wrapping, and race-condition validation under ThreadSanitizer (TSan).

---

### Core Engineering Invariants

* **Linux Kernel Internals:** Namespaces (`CLONE_NEWUSER`, `CLONE_NEWNS`, `CLONE_NEWPID`, `CLONE_NEWNET`), Landlock LSM (ABI v1–v5), Seccomp-BPF, cgroups v2 (`memory.max`, `memory.swap.max=0`, `pids.max`), VFS `pivot_root(2)`, `epoll_pwait2`, `timerfd`, `signalfd4`, PTY line discipline.
* **Low-Level Systems (C++17 / C++20):** Bare-metal systems programming (`-fno-exceptions`, `-fno-rtti`, `-O2`), lockless Sequence Locks (Seqlock), persistent file descriptors, zero heap allocations in execution hot loops, POSIX shared memory (`/dev/shm`).
* **AI Infrastructure & Acceleration:** Local LLM deployment on AMD RDNA 4 (`gfx1201`), ROCm/HIP and Vulkan RADV cooperative matrices, FlashAttention-2 buffer optimization, quantized KV caching (`q8_0`), prompt routing, slot lifecycle management.
* **DevSecOps & Software Supply Chain:** Rootless process sandboxing, linker interception (`-Wl,--wrap`), Clang Sanitizers (`ASan`, `UBSan`, `TSan`), SBOM generation (CycloneDX 1.6 / SPDX 3.0), SCA/OSA dependency quarantine, and mitigation of dependency confusion attacks.

---

### Contact & Artifacts
* **GitHub:** [https://github.com/BannedForWhat](https://github.com/BannedForWhat)
* **Specialization:** Linux Platform Engineering / AI Infrastructure / DevSecOps / SRE
