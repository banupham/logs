# Current Status — Android Cuttlefish on WSL2

Last updated: 2026-08-19 13:47 +07

## Goal

Khởi chạy Android Cuttlefish trên Windows + WSL2, mở thiết bị ảo và chạy smoke test qua `adb`.

## Environment

- Windows host, Ubuntu 26.04 LTS trong WSL2.
- WSL kernel quan sát được: `6.18.33.2-microsoft-standard-WSL2`.
- Architecture: `amd64` / `x86_64`.
- Repo Cuttlefish đã clone tại `~/src/android-cuttlefish`.
- Commit đang checkout: `a318f97e7`.
- Git state: detached HEAD (`HEAD (no branch)`).
- Bazel đã cài: `bazel 9.2.0`, Debian package trạng thái `install ok installed`.
- WSL đã được cập nhật lên version `2.7.12` theo output PowerShell ngày 2026-08-19.

## Đã làm

1. Clone `https://github.com/google/android-cuttlefish.git`.
2. Đã thử build package thủ công bằng tooling của repo.
3. `mk-build-deps` đã tạo `base/cuttlefish-common-build-deps_1.57.0_amd64.deb` nhưng không cài được build dependencies nên build dừng trước `debuild`.
4. Đã kiểm tra output `.deb`: chưa có `cuttlefish-base_*.deb` và chưa có `cuttlefish-user_*.deb`.
5. Đã kiểm tra installed packages: chưa có package Cuttlefish nào được cài.
6. Đã tạo `~/cf`, nhưng thư mục hiện trống; chưa tải `cvd-host_package.tar.gz` và chưa có Android image.
7. `cvd` chưa có trong PATH.
8. Lần kiểm tra virtualization trước cho thấy `/dev/kvm` chưa tồn tại và `vmx|svm` chưa được expose.
9. Đã xác nhận user hiện chưa thuộc các group cần thiết; `cvdnetwork` chưa tồn tại ở thời điểm kiểm tra.
10. Đã chạy thành công từ Windows PowerShell: `wsl --update` và `wsl --shutdown`.
11. PowerShell báo cập nhật WSL thành công lên `2.7.12`.

## Build dependencies còn thiếu

`dpkg-checkbuilddeps` báo thiếu:

- cmake
- dh-exec
- libaom-dev
- libcap-dev
- libclang-dev
- libcurl4-openssl-dev
- libfmt-dev
- libgflags-dev
- libgoogle-glog-dev
- libgtest-dev
- libjsoncpp-dev
- liblzma-dev
- libopus-dev
- libprotobuf-c-dev
- libprotobuf-dev
- libsrtp2-dev
- libssl-dev
- libwayland-dev
- libxml2
- libxml2-dev
- libz3-dev
- protobuf-compiler
- uuid-dev

## Current blockers

### Blocker 1 — KVM / nested virtualization

Sau khi đã chạy `wsl --update` và `wsl --shutdown`, cần mở lại Ubuntu và kiểm tra xem virtualization flags và `/dev/kvm` đã xuất hiện chưa.

### Blocker 2 — Host packages chưa sẵn sàng

Chưa cài `cuttlefish-base` / `cuttlefish-user`, và cũng chưa có host package + Android image để chạy `launch_cvd`.

## Decision / direction

Vì mục tiêu là **mở thiết bị ảo và test nhanh**, ưu tiên dùng package/artifact chính thức thay vì tiếp tục build source thủ công, trừ khi cần debug build riêng.

## Next step

Từ **Windows PowerShell**, mở lại Ubuntu bằng:

```powershell
wsl
```

Sau khi prompt chuyển sang dạng Linux như `duong@DESKTOP-Q1DQINK:~$`, chạy trong **Ubuntu/WSL terminal**:

```bash
grep -c -w 'vmx\|svm' /proc/cpuinfo
ls -l /dev/kvm
lsmod | grep '^kvm'
```

Gửi nguyên output của 3 lệnh kiểm tra KVM để xác định bước tiếp theo.

## Important note

- `wsl --update`, `wsl --shutdown`, và lệnh `wsl` để mở distro chạy từ Windows PowerShell hoặc CMD.
- Các lệnh kiểm tra `/proc/cpuinfo`, `/dev/kvm`, `lsmod` chạy trong Ubuntu/WSL terminal.
