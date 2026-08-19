# Current Status — Android Cuttlefish on WSL2

Last updated: 2026-08-19 14:14 +07

## Goal
Khởi chạy Android Cuttlefish trên Windows + WSL2, mở thiết bị ảo và chạy smoke test qua `adb`.

## Environment
- Windows 10 Pro for Workstations, build `19045.6456`.
- Ubuntu 26.04 LTS trong WSL2.
- WSL distro name: `Ubuntu`.
- WSL version: `2.7.12.0` (lifted/Store package).
- WSL kernel: `6.18.33.2-2`.
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

=> Có safety net trước các thay đổi WSL runtime. Không chạy `wsl --unregister Ubuntu`.

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

## Store package / inbox readiness mới nhất
PowerShell xác nhận:

```text
Package: MicrosoftCorporationII.WindowsSubsystemForLinux_2.7.12.0_x64__8wekyb3d8bbwe
NonRemovable: False
Status: Ok
User duong: Installed
```

Services:
```text
LxssManager  Stopped  Manual
WslService   Running  Automatic
```

Inbox kernel check:
```text
C:\Windows\System32\lxss\tools\kernel
=> không tồn tại / không trả về kết quả
```

Diễn giải:
- Store/lifted WSL package có thể gỡ (`NonRemovable=False`).
- `WslService` đang là active lifted service; `LxssManager` inbox đang tồn tại nhưng stopped.
- Hai optional feature của inbox WSL2 đều Enabled.
- **Inbox kernel chưa được cài**, nên chưa gỡ Store package ngay để tránh rơi vào trạng thái không có kernel WSL2.

Microsoft manual-install docs và troubleshooting ghi rõ kernel MSI cho inbox WSL2 cài kernel vào Windows OS image; nếu `%SystemRoot%\System32\lxss\tools\kernel` thiếu thì cần cài kernel update package.

## Cuttlefish requirement
Cuttlefish cần KVM; `vmx|svm` phải xuất hiện trong `/proc/cpuinfo` và `/dev/kvm` phải tồn tại trước khi launch.

Tham chiếu:
- https://source.android.com/docs/devices/cuttlefish/get-started

## Build state
Build host package từng dừng ở `mk-build-deps` do thiếu dependencies; chưa tạo/cài `cuttlefish-base` / `cuttlefish-user`; `~/cf` chưa có host package hay Android image. Build chưa phải ưu tiên cho tới khi KVM hoạt động.

## Current blocker
Lifted/Store WSL `2.7.12` trên Windows 10 không expose nested virtualization. Hướng đang thử là chuyển về inbox WSL2, nhưng cần cài inbox kernel trước khi gỡ Store runtime.

## Next step
Cài **inbox WSL2 kernel MSI trước**, chưa uninstall Store WSL ở bước này. PowerShell Administrator:

```powershell
wsl --shutdown

$msi = "$env:TEMP\wsl_update_x64.msi"
Invoke-WebRequest -Uri "https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi" -OutFile $msi
Start-Process msiexec.exe -Wait -ArgumentList '/i', "`"$msi`"", '/passive', '/norestart'

Get-Item "$env:SystemRoot\System32\lxss\tools\kernel" -ErrorAction SilentlyContinue |
  Select-Object FullName,Length,LastWriteTime
```

Nếu kernel file xuất hiện, bước sau mới là gỡ **chỉ** package `MicrosoftCorporationII.WindowsSubsystemForLinux` (không gỡ distro Ubuntu, không unregister), reboot nếu cần, rồi xác nhận runtime chuyển sang inbox và test `vmx` + `/dev/kvm`.
