# Current Status — Android Cuttlefish on WSL2

Last updated: 2026-08-19 13:51 +07

## Goal

Khởi chạy Android Cuttlefish trên Windows + WSL2, mở thiết bị ảo và chạy smoke test qua `adb`.

## Environment

- Windows host, Ubuntu 26.04 LTS trong WSL2.
- WSL version: `2.7.12`.
- WSL kernel quan sát được trước đó: `6.18.33.2-microsoft-standard-WSL2`.
- Architecture: `amd64` / `x86_64`.
- CPU: `Intel(R) Core(TM) i5-6198DU CPU @ 2.30GHz`.
- Repo Cuttlefish tại `~/src/android-cuttlefish`.
- Commit: `a318f97e7`.
- Git state: detached HEAD.
- Bazel: `9.2.0`.

## Đã làm

1. Clone `https://github.com/google/android-cuttlefish.git`.
2. Đã thử build package thủ công; build dừng ở `mk-build-deps` vì thiếu dependencies.
3. Chưa tạo được `cuttlefish-base_*.deb` / `cuttlefish-user_*.deb`.
4. Chưa cài package Cuttlefish.
5. `~/cf` chưa có host package hoặc Android image.
6. Đã chạy từ Windows PowerShell: `wsl --update` và `wsl --shutdown`.
7. Đã mở lại Ubuntu/WSL và kiểm tra KVM.
8. Đã kiểm tra Windows virtualization state bằng PowerShell.

## KVM check mới nhất

Trong Ubuntu/WSL:

```text
vmx|svm count: 0
/dev/kvm: không tồn tại
kvm modules: không có
```

## Windows virtualization check mới nhất

PowerShell Administrator trả về:

```text
VirtualizationFirmwareEnabled = False
VMMonitorModeExtensions = False
SecondLevelAddressTranslationExtensions = False
VirtualMachinePlatform = Enabled
.wslconfig: nestedVirtualization=true
hypervisorlaunchtype: không có output
```

## Kết luận hiện tại

Blocker chính nằm ở firmware/BIOS: Windows đang báo `VirtualizationFirmwareEnabled=False`. `VirtualMachinePlatform` và cấu hình WSL đã bật, nhưng CPU virtualization chưa được firmware expose cho Windows/WSL.

## Build dependencies còn thiếu

`cmake dh-exec libaom-dev libcap-dev libclang-dev libcurl4-openssl-dev libfmt-dev libgflags-dev libgoogle-glog-dev libgtest-dev libjsoncpp-dev liblzma-dev libopus-dev libprotobuf-c-dev libprotobuf-dev libsrtp2-dev libssl-dev libwayland-dev libxml2 libxml2-dev libz3-dev protobuf-compiler uuid-dev`

## Decision / direction

Ưu tiên bật Intel CPU virtualization trong BIOS/UEFI trước khi tiếp tục Cuttlefish.

## Next step

Chạy trong **Windows PowerShell — Run as Administrator**:

```powershell
Get-CimInstance Win32_ComputerSystem | Select-Object Manufacturer,Model
```

Gửi output để xác định chính xác phím vào BIOS và vị trí tùy chọn Intel Virtualization Technology/VT-x cho đúng máy.
