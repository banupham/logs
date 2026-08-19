# ChatGPT Work Logs

Repo này dùng để lưu trạng thái công việc giữa các cuộc trò chuyện ChatGPT, để phiên sau có thể đọc lại và tiếp tục đúng chỗ.

## Quy ước

- `STATUS.md`: trạng thái mới nhất, việc đã làm, blocker hiện tại và bước tiếp theo.
- `sessions/YYYY-MM-DD-<topic>.md`: log chi tiết từng phiên làm việc.
- Không lưu password, token, API key, cookie hoặc thông tin nhạy cảm vào repo này.

## Cách dùng ở cuộc trò chuyện sau

Yêu cầu ChatGPT đọc repo `banupham/logs`, đặc biệt là `STATUS.md`, rồi tiếp tục từ mục **Next step**.

## Mục tiêu hiện tại

Khởi chạy Android Cuttlefish trên máy Windows/WSL2, mở thiết bị ảo và chạy smoke test qua `adb`.
