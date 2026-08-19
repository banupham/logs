# Current Status — Android Cuttlefish on WSL2

Last updated: 2026-08-19 13:55 +07

## Goal
Khởi chạy Android Cuttlefish trên Windows + WSL2, mở thiết bị ảo và chạy smoke test qua `adb`.

## Environment
- Windows 10 Pro for Workstations, build `19045`.
- Ubuntu 26.04 LTS trong WSL2.
- WSL version: `2.7.12`.
- WSL kernel quan sát được: `6.18.33.2-microsoft-standard-WSL2`.
- Architecture: `amd64` / `x86_64`.
- Laptop: `ASUSTeK COMPUTER INC. X541UV`.
- CPU: `Intel(R) Core(TM) i5-6198DU CPU @ 2.30GHz`.
- BIOS: `American Megatrends Inc. X541UV.301`, 2016-06-27.
- Người dùng xác nhận `Intel Virtualization Technology` trong BIOS đã `Enabled`.
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
8. Đã kiểm tra virtualization state trên Windows.
9. Đã xác định model máy: ASUS X541UV.
10. Đã xác nhận Hyper-V hypervisor của Windows đang chạy.

## KVM check mới nhất trong WSL
```text
vmx|svm count: 0
/dev/kvm: không tồn tại
kvm modules: không có
```

## Windows virtualization / hypervisor check mới nhất
```text
WindowsProductName: Windows 10 Pro for Workstations
WindowsVersion: 2009
OsBuildNumber: 19045
HyperVisorPresent: True
hypervisorlaunchtype: Auto
VirtualMachinePlatform: Enabled
.wslconfig: nestedVirtualization=true
systeminfo: A hypervisor has been detected
```

Windows có nhiều Hyper-V virtual Ethernet adapter, gồm `vEthernet (WSL)` và `vEthernet (Default Switch)`.

## Kết luận hiện tại
- BIOS virtualization đã được người dùng xác nhận bật.
- Windows Hyper-V hypervisor thực sự đang chạy (`HyperVisorPresent=True`, `hypervisorlaunchtype=Auto`).
- Vì vậy kết quả WMI trước đó `VirtualizationFirmwareEnabled=False` không còn được dùng làm bằng chứng rằng BIOS đang tắt virtualization.
- Blocker hiện tại hẹp hơn: WSL2 chưa nhận nested virtualization (`vmx` không xuất hiện, `/dev/kvm` không có) dù `.wslconfig` đặt `nestedVirtualization=true`.
- Bước chẩn đoán tiếp theo là kiểm tra Virtualization-Based Security / Device Guard / Credential Guard, vì các chính sách bảo mật này có thể liên quan đến nested virtualization.

## Build dependencies còn thiếu
`cmake dh-exec libaom-dev libcap-dev libclang-dev libcurl4-openssl-dev libfmt-dev libgflags-dev libgoogle-glog-dev libgtest-dev libjsoncpp-dev liblzma-dev libopus-dev libprotobuf-c-dev libprotobuf-dev libsrtp2-dev libssl-dev libwayland-dev libxml2 libxml2-dev libz3-dev protobuf-compiler uuid-dev`

## Next step
Chạy trong **Windows PowerShell — Run as Administrator**:

```powershell
Get-CimInstance -ClassName Win32_DeviceGuard -Namespace root\Microsoft\Windows\DeviceGuard | Select-Object VirtualizationBasedSecurityStatus,SecurityServicesConfigured,SecurityServicesRunning,AvailableSecurityProperties,RequiredSecurityProperties
```

Gửi nguyên output để xác định VBS / Device Guard / Credential Guard có đang hoạt động hay không.
