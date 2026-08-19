# Current Status — Android Cuttlefish on Windows

Last updated: 2026-08-19 14:18 +07

## Goal
Khởi chạy Android Cuttlefish trên máy Windows, mở thiết bị ảo và chạy smoke test qua `adb`.

## Environment
- Windows 10 Pro for Workstations, build `19045.6456`.
- CPU: Intel Core i5-6198DU; BIOS VT-x Enabled.
- Hyper-V hypervisor đang chạy.
- Ubuntu distro hiện tại trong WSL2: `Ubuntu`.
- Store/lifted WSL: `2.7.12.0`, kernel `6.18.33.2-2`.
- Repo Cuttlefish trong WSL: `~/src/android-cuttlefish`, commit `a318f97e7`.

## Backup WSL đã xác nhận
```text
C:\Users\duong\wsl-backup.tar
2520647680 bytes
19/08/2026 2:08:43 PM
```
`tar -tf` đọc được rootfs. Không chạy `wsl --unregister Ubuntu`.

## KVM trong Store WSL hiện tại
```text
vmx|svm count: 0
/dev/kvm: không tồn tại
```

## Root cause
WSL lifted/Store trên Windows 10 hiện tự disable nested virtualization. Source WSL 2.7.12 đặt `EnableNestedVirtualization=false` khi host không phải Windows 11+, dù `.wslconfig` có `nestedVirtualization=true`.

Tham chiếu:
- https://github.com/microsoft/WSL/blob/2.7.12/src/windows/service/exe/WslCoreVm.cpp
- https://github.com/microsoft/WSL/issues/40735

Issue #40735 mô tả đây là regression so với inbox WSL2 cũ trên Windows 10.

## Inbox WSL workaround đã thử và dừng
- `Microsoft-Windows-Subsystem-Linux`: Enabled.
- `VirtualMachinePlatform`: Enabled.
- Đã cài legacy inbox kernel MSI thành công:
  `C:\Windows\System32\lxss\tools\kernel` (70722784 bytes).
- Store WSL package là removable.
- Tuy nhiên lệnh được đề xuất trước đó:
  `wsl --update --inbox`
  **không được WSL 2.7.12 hỗ trợ** và trả về:

```text
Invalid command line argument: --inbox
Error code: Wsl/UpdatePackage/E_INVALIDARG
```

=> Hướng dẫn trước về `wsl --update --inbox` là sai cho client này. Không tiếp tục tháo Store WSL / chuyển runtime để tránh làm phức tạp môi trường.

## Cuttlefish requirement
Android docs yêu cầu host Linux thấy virtualization extensions (`vmx` hoặc `svm`) và KVM khả dụng (`/dev/kvm`).

Tham chiếu:
- https://source.android.com/docs/devices/cuttlefish/get-started

## Quyết định kỹ thuật
**Dừng cố chạy Cuttlefish trong WSL2 trên Windows 10 Store WSL.**

Chuyển sang Ubuntu chạy trong **Hyper-V VM riêng**. Microsoft hỗ trợ nested virtualization trên Windows 10 host với Intel VT-x + EPT; VM phải tắt rồi bật:

```powershell
Set-VMProcessor -VMName "Cuttlefish-Ubuntu" -ExposeVirtualizationExtensions $true
```

Tham chiếu:
- https://learn.microsoft.com/windows-server/virtualization/hyper-v/enable-nested-virtualization

Đây là đường trực tiếp để Ubuntu guest thấy VT-x, sau đó dùng KVM cho Cuttlefish.

## Next step
1. Không chạy thêm lệnh gỡ/chuyển WSL.
2. Tạo Hyper-V Generation 2 VM tên chính xác `Cuttlefish-Ubuntu` và cài Ubuntu Linux.
3. Khi VM đang Off, chạy:

```powershell
Set-VMProcessor -VMName "Cuttlefish-Ubuntu" -ExposeVirtualizationExtensions $true
```

4. Boot Ubuntu VM và kiểm tra:

```bash
grep -c -w 'vmx\|svm' /proc/cpuinfo
ls -l /dev/kvm
```

5. Chỉ khi `vmx > 0` và `/dev/kvm` tồn tại mới quay lại build/install Cuttlefish host package và image.
