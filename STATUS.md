# Current Status — Android Cuttlefish on WSL2

Last updated: 2026-08-19 14:06 +07

## Goal
Khởi chạy Android Cuttlefish trên Windows + WSL2, mở thiết bị ảo và chạy smoke test qua `adb`.

## Environment
- Windows 10 Pro for Workstations, build `19045`.
- Ubuntu 26.04 LTS trong WSL2.
- WSL distro name: `Ubuntu`.
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
10. Đã kiểm tra source đúng tag `microsoft/WSL` `2.7.12` và xác nhận WSL lifted/Store tự tắt nested virtualization trên Windows 10.
11. `wsl --list --verbose` xác nhận distro thực tế là `Ubuntu`, đang chạy WSL2.
12. Lệnh export đầu tiên thất bại vì copy nguyên placeholder `<DistroName>` vào PowerShell; chưa có thay đổi phá huỷ nào xảy ra.
13. `Set-VMProcessor -VMName "<VMName>" ...` cũng thất bại vì `<VMName>` chỉ là placeholder và hiện chưa có Hyper-V VM riêng; lệnh này không áp dụng cho distro WSL `Ubuntu`.

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
- VBS có thể cản nested virtualization trong một số Hyper-V scenario, nhưng **không còn là blocker chính cần thử trước** trong cấu hình hiện tại.

## Root cause mới xác định
Source của chính WSL tag `2.7.12` tại `src/windows/service/exe/WslCoreVm.cpp` có logic:

```cpp
if (wsl::windows::common::helpers::IsWindows11OrAbove()) {
    // query NestedVirt support
} else {
    m_vmConfig.EnableNestedVirtualization = false;
}
```

Nghĩa là trên Windows 10, bản WSL lifted/Store `2.7.12` sẽ không expose virtualization extensions cho WSL2 dù `.wslconfig` có `nestedVirtualization=true`.

Tham chiếu:
- https://github.com/microsoft/WSL/blob/2.7.12/src/windows/service/exe/WslCoreVm.cpp
- https://github.com/microsoft/WSL/issues/40735

Điều này khớp chính xác với triệu chứng hiện tại: `vmx=0` và không có `/dev/kvm`.

## Cuttlefish requirement
Tài liệu Android hiện tại yêu cầu host có KVM; `grep -c -w "vmx\|svm" /proc/cpuinfo` phải trả về giá trị khác 0 và `/dev/kvm` phải tồn tại.

Tham chiếu:
- https://source.android.com/docs/devices/cuttlefish/get-started

## Current blocker
**Windows 10 + lifted/Store WSL `2.7.12` không expose nested virtualization.** Vì vậy Cuttlefish không thể launch bằng KVM trong WSL2 hiện tại.

Tắt VBS có thể vẫn cần trong một cấu hình nested virtualization khác, nhưng với WSL `2.7.12` trên Windows 10 thì source WSL đã chặn trước đó, nên không nên reboot/tắt VBS chỉ để thử bước này.

## Build dependencies còn thiếu
`cmake dh-exec libaom-dev libcap-dev libclang-dev libcurl4-openssl-dev libfmt-dev libgflags-dev libgoogle-glog-dev libgtest-dev libjsoncpp-dev liblzma-dev libopus-dev libprotobuf-c-dev libprotobuf-dev libsrtp2-dev libssl-dev libwayland-dev libxml2 libxml2-dev libz3-dev protobuf-compiler uuid-dev`

## Next step
Backup distro `Ubuntu` trước mọi thay đổi WSL. Chạy trong Windows PowerShell:

```powershell
wsl --terminate Ubuntu
wsl --export Ubuntu "$env:USERPROFILE\wsl-backup.tar"
Get-Item "$env:USERPROFILE\wsl-backup.tar" | Select-Object FullName,Length,LastWriteTime
```

Có thể kiểm tra archive đọc được bằng:

```powershell
tar -tf "$env:USERPROFILE\wsl-backup.tar" | Select-Object -First 20
```

Không chạy `wsl --unregister` cho tới khi export thành công và archive đã được kiểm tra.

`Set-VMProcessor` chỉ dùng nếu sau này tạo **Hyper-V VM riêng**; không dùng tên `Ubuntu` của distro WSL cho lệnh đó.

Sau khi backup được xác nhận, bước tiếp theo mới là đánh giá cách chuyển/test inbox WSL2 của Windows 10 và kiểm tra lại KVM.
