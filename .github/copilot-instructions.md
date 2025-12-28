# AI Coding Agent Instructions for microkernelia

## Big Picture
- Rust no_std unikernel that boots, initializes memory and virtio devices, then serves MCP over virtio-vsock. AI runtime is an in-kernel stub that loads a tiny model via virtio-fs.
- All guest interactions are MCP messages framed as length-prefixed JSON over vsock; host-side tests use `tools/mcp-cli`.

## Workspace Roles
- Kernel: [kernel](kernel) — entry, MMU, cooperative scheduler, serial logging.
- Virtio drivers: [drivers_virtio](drivers_virtio) — vsock and fs queues, BAR mapping.
- MCP core: [mcp_core](mcp_core) — MCP tools/dispatch and JSON-RPC stubs.
- MCP transport: [mcp_vsock_transport](mcp_vsock_transport) — frame read/write over vsock.
- AI runtime: [ai_runtime](ai_runtime) — `load_model()` via fs; `infer()` over in-memory table.
- Logging: [logging](logging) — ring buffer `log_write()`/`log_read()` used by `serial_println!`.
- Tooling: [xtask](xtask) build/run helpers; [tools/mcp-cli](tools/mcp-cli) host client.

## Build & Run
- Full workspace: `cargo build --workspace --release` (see [BUILD.md](BUILD.md)).
- Bare-metal kernel: `cargo build -p kernel --target x86_64-unknown-none --release --features global-allocator`.
- Cargo Make tasks in [Makefile.toml](Makefile.toml):
  - `cargo make build-all` — kernel + tools.
  - `cargo make link-elf` — links `target/kernel.elf` with `ld.lld` using [kernel/link.ld](kernel/link.ld).
  - `cargo make image-iso` — Limine-based ISO scaffold (requires local `limine/*`).
  - `cargo make qemu` — runs QEMU via [xtask/src/main.rs](xtask/src/main.rs).
- Windows note: `tools/mcp-cli` uses TCP fallback (`127.0.0.1:5000`); on Unix connects to `/tmp/vm.sock`.

## Runtime & IPC Patterns
- Vsock framing in [mcp_vsock_transport/src/lib.rs](mcp_vsock_transport/src/lib.rs):
  - `read_frame(buf)` reads `u32` big-endian length + payload; caps at 1 MiB.
  - `write_frame(json)` sends length-prefixed JSON via `drivers_virtio::vsock::send`.
- Virtio vsock in [drivers_virtio/src/lib.rs](drivers_virtio/src/lib.rs):
  - `vsock::init()` sets up TX/RX queues and BAR0 mapping via `map_bar0_phys_to_virt()`.
  - `vsock::send(&[u8])` / `vsock::recv(&mut [u8])` are the primary I/O.
- MCP server in [mcp_core/src/lib.rs](mcp_core/src/lib.rs):
  - Tools: `infer`, `health`, `metadata`, `load_model`, `logs` via `mcp_server::dispatch()`.
  - `mcp_server_loop()` pulls frames, parses JSON-RPC, dispatches, and replies.

## Memory & Safety Conventions
- Kernel entry: [kernel/src/lib.rs](kernel/src/lib.rs) `_start()` initializes MMU, guard page, stack canary, and global heap (`linked_list_allocator`).
- Cross-crate FFI uses `extern "Rust"` for kernel-provided allocators: `alloc_frame()`, `alloc_aligned(size, align)`.
- MMU helpers: `mmu_init()`, `mmu_protect_sections(...)`, `map_phys_to_virt(...)`, `map_mmio_region(...)`, `mmu_insert_guard_page(...)`.

## Logging & Observability
- Use `serial_println!("...")` in kernel/driver code; it writes to serial and to the ring buffer via [logging/src/lib.rs](logging/src/lib.rs).
- Expose logs to clients through MCP tool `logs` (`mcp_server::handle_logs`).

## AI Runtime
- `ai_runtime::load_model(path)` reads from virtio-fs into a static buffer; sets `MODEL`.
- `ai_runtime::infer(prompt)` returns a response by scanning the in-memory key/value model.

## Project-Specific Conventions
- `no_std` for kernel and device crates; `std` only in tooling (`xtask`, `mcp-cli`).
- Feature flags:
  - `kernel` (drivers): disable panic handler when used by kernel.
  - `global-allocator` (kernel/mcp_core): enable global allocator setup.
  - `virtio-log` (drivers): enable structured logging of fs/vsock events.
- JSON-RPC parsing/serialization in `mcp_core::ai_stub` are stubs (intended to be implemented).

## Integration Hotspots
- MCP I/O path: `drivers_virtio::vsock` → `mcp_vsock_transport` → `mcp_core::mcp_server`.
- Model loading path: `drivers_virtio::fs` → `ai_runtime::load_model`.

## Gotchas & Tips
- QEMU run in `xtask qemu` expects `target/kernel.elf` from `cargo make link-elf`.
- `drivers_virtio` uses `static mut` for queues/state; treat as single-core cooperative.
- `alloc_frame()` in drivers discards the address; if you need it, use kernel `alloc_frame_get()`.
- Message caps: frames > 1 MiB are rejected.

## What To Implement Next (Agents)
- Fill `mcp_core::ai_stub` parsing/serialization helpers and wire `infer/health/metadata/load_model` requests.
- Add integration tests that boot via `cargo make qemu` and exercise MCP tools via `tools/mcp-cli`.
