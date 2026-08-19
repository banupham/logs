# Current Status — Android Cuttlefish on WSL2

Last updated: 2026-08-19 13:52 +07

## Goal
Khởi chạy Android Cuttlefish trên Windows + WSL2, mở thiết bị ảo và chạy smoke test qua `adb`.

## Environment
- Windows host, Ubuntu 26.04 LTS trong WSL2.
- WSL version: `2.7.12`.
- WSL kernel quan sát được: `6.18.33.2-microsoft-standard-WSL2`.
- Architecture: `amd64` / `x86_64`.
- Laptop: `ASUSTeK COMPUTER INC. X541UV`.
- CPU: `Intel(R) Core(TM) i5-6198DU CPU @ 2.30GHz`.
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

## KVM check mới nhất
```text
vmx|svm count: 0
/dev/kvm: không tồn tại
kvm modules: không có
```

## Windows virtualization check mới nhất
```text
VirtualizationFirmwareEnabled = False
VMMonitorModeExtensions = False
SecondLevelAddressTranslationExtensions = False
VirtualMachinePlatform = Enabled
.wslconfig: nestedVirtualization=true
hypervisorlaunchtype: không có output
```

## Kết luận hiện tại
Blocker chính là virtualization đang chưa được firmware/BIOS expose. Cần bật Intel Virtualization Technology (VT-x/VMX) trong BIOS trước khi tiếp tục Cuttlefish.

## Build dependencies còn thiếu
`cmake dh-exec libaom-dev libcap-dev libclang-dev libcurl4-openssl-dev libfmt-dev libgflags-dev libgoogle-glog-dev libgtest-dev libjsoncpp-dev liblzma-dev libopus-dev libprotobuf-c-dev libprotobuf-dev libsrtp2-dev libssl-dev libwayland-dev libxml2 libxml2-dev libz3-dev protobuf-compiler uuid-dev`

## Next step
Trên ASUS X541UV, vào BIOS/UEFI và bật Intel Virtualization Technology:

1. Từ Windows PowerShell (Administrator) chạy:
```powershell
shutdown /r /fw /t 0
```
2. Trong BIOS, vào `Advanced` → `CPU Configuration`.
3. Đặt `Intel Virtualization Technology` hoặc `Intel (VMX) Virtualization Technology` thành `Enabled`.
4. Nhấn `F10` để Save & Exit.
5. Sau khi Windows khởi động lại, kiểm tra lại trạng thái virtualization trước khi quay lại WSL.

Nguồn tham chiếu: ASUS X541UV support page và ASUS official virtualization BIOS guidance.
