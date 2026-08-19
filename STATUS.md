# Current Status — Android Cuttlefish on WSL2

Last updated: 2026-08-19 14:16 +07

## Goal
Khởi chạy Android Cuttlefish trên Windows + WSL2, mở thiết bị ảo và chạy smoke test qua `adb`.

## Environment
- Windows 10 Pro for Workstations, build `19045.6456`.
- Ubuntu 26.04 LTS trong WSL2.
- WSL distro name: `Ubuntu`.
- WSL version: `2.7.12.0` (lifted/Store package).
- WSL kernel hiện tại khi chạy Store runtime: `6.18.33.2-2`.
- WSLg: `1.0.73.2`.
- Architecture: `amd64` / `x86_64`.
- Laptop: `ASUSTeK COMPUTER INC. X541UV`.
- CPU: `Intel(R) Core(TM) i5-6198DU CPU @ 2.30GHz`.
- BIOS VT-x: Enabled.
- Repo Cuttlefish: `~/src/android-cuttlefish`, commit `a318f97e7` (detached HEAD).
- Bazel: `9.2.0`.

## Backup đã xác nhận
Distro `Ubuntu` đã được terminate và export thành công:

```text
C:\Users\duong\wsl-backup.tar
2520647680 bytes
19/08/2026 2:08:43 PM
```

`tar -tf` đọc được rootfs (`./etc/`, `./usr/`, `./root/`, `./home/`, `./bin/`, ...).

=> Có safety net trước các thay đổi WSL runtime. **Không chạy `wsl --unregister Ubuntu`.**

## KVM check hiện tại
```text
vmx|svm count: 0
/dev/kvm: không tồn tại
kvm modules: không có
```

## Windows / WSL virtualization state
```text
Windows: 10.0.19045.6456
HyperVisorPresent: True
hypervisorlaunchtype: Auto
Microsoft-Windows-Subsystem-Linux: Enabled
VirtualMachinePlatform: Enabled
.wslconfig: nestedVirtualization=true
```

VBS/Device Guard:
```text
VirtualizationBasedSecurityStatus : 2
SecurityServicesConfigured        : {0}
SecurityServicesRunning           : {0}
```

## Root cause
Source của WSL tag `2.7.12` (`src/windows/service/exe/WslCoreVm.cpp`) tự đặt `EnableNestedVirtualization = false` trên Windows 10. Vì vậy lifted/Store WSL 2.7.12 không expose VMX cho guest dù `.wslconfig` có `nestedVirtualization=true`.

Tham chiếu:
- https://github.com/microsoft/WSL/blob/2.7.12/src/windows/service/exe/WslCoreVm.cpp
- https://github.com/microsoft/WSL/issues/40735

Issue #40735 ghi nhận inbox WSL2 trên Windows 10 19041-19045 từng expose nested virtualization, còn lifted WSL hiện chặn nó.

## Store package / inbox readiness
Store package:
```text
MicrosoftCorporationII.WindowsSubsystemForLinux_2.7.12.0_x64__8wekyb3d8bbwe
NonRemovable: False
Status: Ok
User duong: Installed
```

Services trước chuyển đổi:
```text
LxssManager  Stopped  Manual
WslService   Running  Automatic
```

Hai Windows optional feature cần cho inbox WSL2 đều `Enabled`:
- `Microsoft-Windows-Subsystem-Linux`
- `VirtualMachinePlatform`

## Inbox kernel đã cài thành công
Đã download và cài Microsoft x64 WSL2 kernel update MSI bằng `msiexec`.

Sau cài đặt:
```text
FullName: C:\Windows\System32\lxss\tools\kernel
Length: 70722784
LastWriteTime: 13/04/2021 12:06:16 PM
```

=> Inbox WSL2 kernel hiện đã tồn tại; không còn blocker “thiếu kernel” trước khi tháo Store/lifted runtime.

Microsoft docs mô tả `wsl --update --inbox` là cập nhật riêng inbox WSL2 kernel, không cài Store version.

## Cuttlefish requirement
Cuttlefish cần KVM; `vmx|svm` phải xuất hiện trong `/proc/cpuinfo` và `/dev/kvm` phải tồn tại trước khi launch.

Tham chiếu:
- https://source.android.com/docs/devices/cuttlefish/get-started

## Build state
Build host package từng dừng ở `mk-build-deps` do thiếu dependencies; chưa tạo/cài `cuttlefish-base` / `cuttlefish-user`; `~/cf` chưa có host package hay Android image. Build chưa phải ưu tiên cho tới khi KVM hoạt động.

## Current blocker
Lifted/Store WSL `2.7.12` trên Windows 10 không expose nested virtualization. Inbox prerequisites + kernel hiện đã sẵn sàng, nên bước kế tiếp là chuyển runtime từ Store/lifted sang inbox.

## Next step
PowerShell Administrator:

```powershell
wsl --shutdown
wsl --update --inbox
```

Nếu `wsl --update --inbox` thành công, gỡ **chỉ Store WSL core package**:

```powershell
$pkg = Get-AppxPackage -AllUsers -Name MicrosoftCorporationII.WindowsSubsystemForLinux
$pkg | Select-Object Name,PackageFullName,Version,NonRemovable
Remove-AppxPackage -Package $pkg.PackageFullName -AllUsers
Restart-Computer
```

**Không chạy `wsl --unregister Ubuntu`. Không uninstall Ubuntu app.**

Sau reboot, kiểm tra Windows side:

```powershell
Get-AppxPackage -AllUsers MicrosoftCorporationII.WindowsSubsystemForLinux
Get-Service LxssManager,WslService -ErrorAction SilentlyContinue | Select Name,Status,StartType
wsl --status
wsl -l -v
```

Sau đó launch `Ubuntu` và kiểm tra Linux side:

```bash
uname -r
grep -c -w 'vmx\|svm' /proc/cpuinfo
ls -l /dev/kvm
```

Nếu `vmx > 0` và `/dev/kvm` tồn tại, quay lại cài/build Cuttlefish host package và image. Nếu vẫn `vmx=0`, đánh giá VBS/CPU/inbox nested virtualization trước khi fallback sang Hyper-V VM riêng.
