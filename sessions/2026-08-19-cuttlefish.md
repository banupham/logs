# Session — 2026-08-19 — Android Cuttlefish on WSL2

## Context
Tiếp tục từ `STATUS.md` với blocker: Windows 10 host, WSL `2.7.12`, `.wslconfig` có `nestedVirtualization=true`, BIOS VT-x bật, Hyper-V chạy, nhưng trong WSL vẫn `vmx=0` và không có `/dev/kvm`.

## Việc đã kiểm tra
- Đọc source của chính Microsoft WSL tag `2.7.12`.
- Tại `src/windows/service/exe/WslCoreVm.cpp`, khi `EnableNestedVirtualization` được yêu cầu, WSL chỉ query/expose `NestedVirt` nếu `IsWindows11OrAbove()`; nhánh Windows 10 đặt `m_vmConfig.EnableNestedVirtualization = false`.
- Vì vậy cấu hình `.wslconfig` không thể thắng logic này trên lifted/Store WSL 2.7.12 chạy Windows 10.
- Triệu chứng quan sát (`vmx=0`, `/dev/kvm` không tồn tại) khớp với logic trên.
- Tài liệu Cuttlefish yêu cầu KVM, nên tiếp tục build host package lúc này chưa giải quyết được blocker launch.

## Kết luận
Không nên ưu tiên tắt VBS/reboot như kế hoạch cũ. VBS có thể ảnh hưởng nested virtualization ở một số Hyper-V scenario, nhưng với cấu hình này WSL 2.7.12 đã chủ động tắt nested virtualization trên Windows 10 trước đó.

## Hướng tiếp theo
1. Backup distro WSL trước mọi thao tác thay đổi runtime:
   ```powershell
   wsl --list --verbose
   wsl --export <DistroName> "$env:USERPROFILE\wsl-backup.tar"
   ```
2. Đánh giá khả năng quay về/test inbox WSL2 của Windows 10, do báo cáo upstream cho thấy khác biệt giữa inbox WSL và lifted/Store WSL về nested virtualization trên Windows 10.
3. Sau khi đổi runtime, kiểm tra lại:
   ```bash
   grep -c -w 'vmx\|svm' /proc/cpuinfo
   ls -l /dev/kvm
   ```
4. Nếu inbox WSL không khả thi hoặc vẫn không expose KVM, fallback là Ubuntu VM trên Hyper-V với `ExposeVirtualizationExtensions`, rồi chạy Cuttlefish trong VM đó.

## References
- Microsoft WSL 2.7.12 source: https://github.com/microsoft/WSL/blob/2.7.12/src/windows/service/exe/WslCoreVm.cpp
- WSL issue: https://github.com/microsoft/WSL/issues/40735
- Android Cuttlefish get started: https://source.android.com/docs/devices/cuttlefish/get-started
