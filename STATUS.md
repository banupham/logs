# Current Status — Android Cuttlefish on WSL2

Last updated: 2026-08-19 13:56 +07

## Goal
Khởi chạy Android Cuttlefish trên Windows + WSL2, mở thiết bị ảo và chạy smoke test qua `adb`.

## Environment
- Windows 10 Pro for Workstations, build `19045`.
- Ubuntu 26.04 LTS trong WSL2.
- WSL version: `2.7.12`.
- WSL kernel: `6.18.33.2-microsoft-standard-WSL2`.
- Architecture: `amd64` / `x86_64`.
- Laptop: `ASUSTeK COMPUTER INC. X541UV`.
- CPU: `Intel(R) Core(TM) i5-6198DU CPU @ 2.30GHz`.
- BIOS: `American Megatrends Inc. X541UV.301`, 2016-06-27.
- `Intel Virtualization Technology` trong BIOS đã `Enabled`.
- Repo Cuttlefish: `~/src/android-cuttlefish`.
- Commit: `a318f97e7` (detached HEAD).
- Bazel: `9.2.0`.

## Đã làm
1. Clone `https://github.com/google/android-cuttlefish.git`.
2. Thử build package thủ công; build dừng ở `mk-build-deps` vì thiếu dependencies.
3. Chưa tạo được `cuttlefish-base_*.deb` / `cuttlefish-user_*.deb`.
4. Chưa cài package Cuttlefish.
5. `~/cf` chưa có host package hoặc Android image.
6. Đã chạy `wsl --update` và `wsl --shutdown` từ Windows PowerShell.
7. Đã kiểm tra lại KVM trong WSL.
8. Đã xác nhận Hyper-V hypervisor của Windows đang chạy.
9. Đã kiểm tra VBS / Device Guard.

## KVM check mới nhất trong WSL
```text
vmx|svm count: 0
/dev/kvm: không tồn tại
kvm modules: không có
```

## Windows virtualization / hypervisor
```text
WindowsProductName: Windows 10 Pro for Workstations
OsBuildNumber: 19045
HyperVisorPresent: True
hypervisorlaunchtype: Auto
VirtualMachinePlatform: Enabled
.wslconfig: nestedVirtualization=true
systeminfo: A hypervisor has been detected
```

## VBS / Device Guard check mới nhất
PowerShell Administrator:
```text
VirtualizationBasedSecurityStatus : 2
SecurityServicesConfigured        : {0}
SecurityServicesRunning           : {0}
AvailableSecurityProperties       : {1, 3}
RequiredSecurityProperties        : {0}
```

Diễn giải:
- `VirtualizationBasedSecurityStatus = 2`: VBS đang enabled và running.
- Không có Credential Guard / Memory Integrity service đang configured hoặc running (`{0}`).
- Microsoft Hyper-V nested virtualization troubleshooting ghi nhận VBS/Device Guard active có thể chặn nested virtualization.

## Current blocker
WSL2 chưa nhận nested virtualization (`vmx` không xuất hiện, `/dev/kvm` không có) dù BIOS VT-x đã bật, Hyper-V đang chạy và `.wslconfig` đã `nestedVirtualization=true`.

## Build dependencies còn thiếu
`cmake dh-exec libaom-dev libcap-dev libclang-dev libcurl4-openssl-dev libfmt-dev libgflags-dev libgoogle-glog-dev libgtest-dev libjsoncpp-dev liblzma-dev libopus-dev libprotobuf-c-dev libprotobuf-dev libsrtp2-dev libssl-dev libwayland-dev libxml2 libxml2-dev libz3-dev protobuf-compiler uuid-dev`

## Next step
Tạm tắt VBS để kiểm tra nested virtualization. Thực hiện trong **Windows PowerShell — Run as Administrator**:

```powershell
reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows\DeviceGuard" /v EnableVirtualizationBasedSecurity /t REG_DWORD /d 0 /f
reg add "HKLM\SYSTEM\CurrentControlSet\Control\DeviceGuard" /v EnableVirtualizationBasedSecurity /t REG_DWORD /d 0 /f
bcdedit /set vsmlaunchtype off
Restart-Computer
```

Sau reboot sẽ kiểm tra lại VBS và KVM. Không tắt `hypervisorlaunchtype`, vì WSL2 vẫn cần Hyper-V hypervisor.
