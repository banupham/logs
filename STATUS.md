# Current Status — Android Cuttlefish on WSL2

Last updated: 2026-08-19 13:42 +07

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

## Đã làm

1. Clone `https://github.com/google/android-cuttlefish.git`.
2. Đã thử build package thủ công bằng tooling của repo.
3. `mk-build-deps` đã tạo `base/cuttlefish-common-build-deps_1.57.0_amd64.deb` nhưng không cài được build dependencies nên build dừng trước `debuild`.
4. Đã kiểm tra output `.deb`: chưa có `cuttlefish-base_*.deb` và chưa có `cuttlefish-user_*.deb`.
5. Đã kiểm tra installed packages: chưa có package Cuttlefish nào được cài.
6. Đã tạo `~/cf`, nhưng thư mục hiện trống; chưa tải `cvd-host_package.tar.gz` và chưa có Android image.
7. `cvd` chưa có trong PATH.
8. Đã kiểm tra virtualization trong WSL: `/dev/kvm` chưa tồn tại và `vmx|svm` chưa được expose ở lần kiểm tra gần nhất.
9. Đã xác nhận user hiện chưa thuộc các group cần thiết; `cvdnetwork` chưa tồn tại ở thời điểm kiểm tra.

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

Lần kiểm tra gần nhất trong WSL:

```bash
grep -c -w 'vmx\|svm' /proc/cpuinfo
ls -l /dev/kvm
```

cho thấy không có virtualization flags và không có `/dev/kvm`.

Đây là blocker quan trọng nhất để thực sự launch Cuttlefish.

### Blocker 2 — Host packages chưa sẵn sàng

Chưa cài `cuttlefish-base` / `cuttlefish-user`, và cũng chưa có host package + Android image để chạy `launch_cvd`.

## Decision / direction

Vì mục tiêu là **mở thiết bị ảo và test nhanh**, ưu tiên dùng package/artifact chính thức thay vì tiếp tục build source thủ công, trừ khi cần debug build riêng.

## Next step

Thực hiện từ **Windows PowerShell**, không phải Ubuntu shell:

1. Kiểm tra/chỉnh `%USERPROFILE%\.wslconfig` để đảm bảo:

```ini
[wsl2]
nestedVirtualization=true
```

2. Chạy:

```powershell
wsl --update
wsl --shutdown
```

3. Mở lại Ubuntu rồi chạy:

```bash
grep -c -w 'vmx\|svm' /proc/cpuinfo
ls -l /dev/kvm
lsmod | grep '^kvm'
```

4. Nếu `/dev/kvm` xuất hiện và flags > 0, đi tiếp:
   - cài `cuttlefish-base` + `cuttlefish-user` từ Artifact Registry,
   - add user vào `kvm,cvdnetwork,render`,
   - restart WSL,
   - tải `cvd-host_package.tar.gz` + `aosp_cf_x86_64_phone-img-*.zip` cùng build ID,
   - extract vào `~/cf`,
   - chạy `HOME=$PWD ./bin/launch_cvd --daemon`,
   - kiểm tra `./bin/adb devices`, `sys.boot_completed`, Android version/model/ABI,
   - mở WebRTC UI tại localhost port 8443 nếu hoạt động.

## Important note

Lệnh `wsl --update` / `wsl --shutdown` phải chạy từ Windows PowerShell hoặc CMD. Khi chạy trong Ubuntu shell sẽ báo `Command 'wsl' not found`; không cài package Ubuntu tên `wsl` để xử lý việc này.
