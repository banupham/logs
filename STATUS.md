# Current Status — Android Cuttlefish on WSL2

Last updated: 2026-08-19 13:49 +07

## Goal

Khởi chạy Android Cuttlefish trên Windows + WSL2, mở thiết bị ảo và chạy smoke test qua `adb`.

## Environment

- Windows host, Ubuntu 26.04 LTS trong WSL2.
- WSL version: `2.7.12`.
- WSL kernel quan sát được trước đó: `6.18.33.2-microsoft-standard-WSL2`.
- Architecture: `amd64` / `x86_64`.
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

## KVM check mới nhất

Chạy trong Ubuntu/WSL:

```bash
grep -c -w 'vmx\|svm' /proc/cpuinfo
ls -l /dev/kvm
lsmod | grep '^kvm'
```

Kết quả:

```text
0
ls: cannot access '/dev/kvm': No such file or directory
(no kvm modules listed)
```

Kết luận hiện tại: WSL2 chưa nhận virtualization extensions, nên chưa thể launch Cuttlefish.

## Build dependencies còn thiếu

`cmake dh-exec libaom-dev libcap-dev libclang-dev libcurl4-openssl-dev libfmt-dev libgflags-dev libgoogle-glog-dev libgtest-dev libjsoncpp-dev liblzma-dev libopus-dev libprotobuf-c-dev libprotobuf-dev libsrtp2-dev libssl-dev libwayland-dev libxml2 libxml2-dev libz3-dev protobuf-compiler uuid-dev`

## Current blocker

KVM / nested virtualization chưa hoạt động trong WSL2.

## Decision / direction

Ưu tiên làm cho KVM hoạt động trước. Cuttlefish yêu cầu virtualization/KVM trên host; chưa tiếp tục cài host package hoặc tải Android image cho đến khi xác định nguyên nhân KVM không xuất hiện.

## Next step

Chạy trong **Windows PowerShell — Run as Administrator**:

```powershell
Get-CimInstance Win32_Processor | Select-Object Name,VirtualizationFirmwareEnabled,VMMonitorModeExtensions,SecondLevelAddressTranslationExtensions

Get-WindowsOptionalFeature -Online -FeatureName VirtualMachinePlatform

Get-Content "$env:USERPROFILE\.wslconfig" -ErrorAction SilentlyContinue

bcdedit /enum {current} | findstr /i hypervisorlaunchtype
```

Gửi nguyên output 4 lệnh trên để xác định bước tiếp theo.
