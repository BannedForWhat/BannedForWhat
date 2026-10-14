# Dmitry Galoshin (@BannedForWhat)

**Linux Platform & AI Infrastructure Engineer**  
Focused on low-level Linux systems programming, deterministic C++17 memory models, kernel DRM/TTM subsystems, and resilient L7 AI gateways.

---

### Architecture: Sentinel-Mesh Inference Infrastructure

```text
 [ Physical AMD RDNA 2 GPU / Linux Kernel 6.x ]
                       │
                       ▼ pread(fd, buf, 64, 0)
    +─────────────────────────────────────+
    |         amdgpu-sentinel             |  • C++17, epoll_pwait2, timerfd, signalfd4
    |    (Daemon: 420 KB RSS, cgroups v2) |  • Zero-alloc hot path, lockless VFS read
    +──────────────────┬──────────────────+
                       │
                       ▼ POSIX SHM (mmap, Seqlock <100 ns)
              /dev/shm/vram_guard
                       ▲
                       │ Zero-copy telemetry read
    +──────────────────┴──────────────────+
    |           prompt-mesh               |  • Python 3.12 asyncio, RSS ~13 MB
    |     (L7 Gateway: OpenAI API :8000)  |  • Middle-Out Pruning (anti-TTM spillover)
    +──────────────────┬──────────────────+  • Active cancellation on client abort
                       │
                       ▼ Proxy to local cluster
         llama-server (:8080 / :8082)

 ───────────────────────────────────────────────────────────────────────────
  Testing & DevSecOps: [sentinel-mesh-ops]
  • Headless Bubblewrap (bwrap) sysfs mocking in GitHub Actions (Ubuntu 24.04)
  • Zero-Allocation proof via GNU ld linker wrapping: -Wl,--wrap=malloc
  • Multi-compiler sanitizer matrix: ASan / UBSan / TSan (Seqlock verification)
mkdir -p ~/BannedForWhat
cd ~/BannedForWhat
git init
git branch -M main

cat << 'EOF' > README.md
# Dmitry Galoshin (@BannedForWhat)

**Linux Platform & AI Infrastructure Engineer**  
Focused on low-level Linux systems programming, deterministic C++17 memory models, kernel DRM/TTM subsystems, and resilient L7 AI gateways.

---

### Architecture: Sentinel-Mesh Inference Infrastructure

```text
 [ Physical AMD RDNA 2 GPU / Linux Kernel 6.x ]
                       │
                       ▼ pread(fd, buf, 64, 0)
    +─────────────────────────────────────+
    |         amdgpu-sentinel             |  • C++17, epoll_pwait2, timerfd, signalfd4
    |    (Daemon: 420 KB RSS, cgroups v2) |  • Zero-alloc hot path, lockless VFS read
    +──────────────────┬──────────────────+
                       │
                       ▼ POSIX SHM (mmap, Seqlock <100 ns)
              /dev/shm/vram_guard
                       ▲
                       │ Zero-copy telemetry read
    +──────────────────┴──────────────────+
    |           prompt-mesh               |  • Python 3.12 asyncio, RSS ~13 MB
    |     (L7 Gateway: OpenAI API :8000)  |  • Middle-Out Pruning (anti-TTM spillover)
    +──────────────────┬──────────────────+  • Active cancellation on client abort
                       │
                       ▼ Proxy to local cluster
         llama-server (:8080 / :8082)

 ───────────────────────────────────────────────────────────────────────────
  Testing & DevSecOps: [sentinel-mesh-ops]
  • Headless Bubblewrap (bwrap) sysfs mocking in GitHub Actions (Ubuntu 24.04)
  • Zero-Allocation proof via GNU ld linker wrapping: -Wl,--wrap=malloc
  • Multi-compiler sanitizer matrix: ASan / UBSan / TSan (Seqlock verification)
```

---

### Verified Repositories

* **[amdgpu-sentinel](https://github.com/BannedForWhat/amdgpu-sentinel)** — Zero-allocation C++17 telemetry daemon for AMD RDNA 2 sysfs (420 KB RSS, systemd cgroups v2).
* **[prompt-mesh](https://github.com/BannedForWhat/prompt-mesh)** — Hardware-aware L7 AI Gateway on pure asyncio, middle-out context pruning, and circuit breaker.
* **[sentinel-mesh-ops](https://github.com/BannedForWhat/sentinel-mesh-ops)** — Automated reliability harness, bwrap sysfs emulation, linker allocation checks, and sanitizer matrices.

### Core Technical Focus
`Linux 6.x` • `C++17 (no-RTTI/no-exceptions)` • `POSIX IPC` • `cgroups v2` • `Seccomp BPF` • `ROCm/HIP` • `Python asyncio` • `DevSecOps / SCA`
