# zarust

[![Rust](https://img.shields.io/badge/rust-1.75%2B-orange.svg)](https://www.rust-lang.org/)
[![Platform](https://img.shields.io/badge/platform-Windows%20%7C%20Linux%20%7C%20macOS-0068FF.svg)](https://github.com/CgTuan210/zarust)
[![License: MIT OR Apache-2.0](https://img.shields.io/badge/license-MIT%20OR%20Apache--2.0-blue.svg)](LICENSE-MIT)

Ứng dụng **Zalo Desktop Native** viết hoàn toàn bằng **Rust** (`egui` cho giao diện đồ họa + `ratatui` cho terminal), giao tiếp trực tiếp với WebSocket và API của Zalo mà **không dùng Electron, Node.js hay Chromium**.

![Giao diện ứng dụng](docs/assets/demo-windows.png)

---

## Điểm nổi bật

- **Siêu nhẹ & Phản hồi tức thì:** Chỉ chạy **1 tiến trình duy nhất**, tiêu thụ khoảng **60 – 160 MB RAM** khi hoạt động (tiết kiệm >85–90% RAM so với bản Electron), mở cửa sổ gần như ngay lập tức.
- **Đồng bộ tin nhắn mượt mà, không khựng giao diện:**
  - Toàn bộ mạng, giải mã và lưu trữ SQLite chạy tách biệt trên luồng nền (`Tokio` + `spawn_blocking`), không bao giờ khóa khung hình UI.
  - Ghi lịch sử theo lô giao dịch (`SQLite WAL` + delta persistence), tải song song đa luồng cho hội thoại cá nhân, Nhóm và *Cloud của tôi*, hỗ trợ đồng bộ lịch sử từ điện thoại (Sync V2/V1) và phân trang tin nhắn cũ.
- **Kiểm soát thông báo chuẩn xác:**
  - Tắt/bật thông báo toàn cục hoặc tắt riêng từng cuộc trò chuyện (1-1 và Nhóm, đồng bộ hai chiều với điện thoại).
  - Phân tách rõ tin nhắn trực tiếp và luồng đồng bộ lịch sử — không bắn pop-up hay phát chuông làm phiền khi đang tải lại tin nhắn cũ.
- **Hai chế độ sử dụng trong cùng 1 binary:** Giao diện desktop hiện đại (`--gui`) hoặc giao diện dòng lệnh gọn nhẹ (`--tui`) cho máy cấu hình yếu / SSH.

---

## Tính năng chính

### Nhắn tin & Quản lý hội thoại
- Đăng nhập bằng **mã QR** hoặc **mật khẩu**; tự động khôi phục phiên từ `session.json`.
- Hỗ trợ **nhiều tài khoản (Multi-Account)**: chuyển đổi tài khoản tức thì và nhận thông báo nền từ các tài khoản phụ.
- Chat 1-1, Nhóm, *Cloud của tôi*; trả lời trích dẫn (*quote*), chuyển tiếp, thả cảm xúc (*reaction*), ghim tin nhắn/hội thoại, đánh dấu sao, thu hồi tin nhắn, tin nhắn tự xóa (*vanish*) và chống thu hồi (*anti-recall*).
- Hẹn giờ gửi tin nhắn tự động, nhắc hẹn (*reminder*) và lưu bản nháp theo từng hội thoại.

### Đa phương tiện, Nhãn dán & Tài liệu
- Gửi ảnh (hỗ trợ dán trực tiếp từ clipboard `Ctrl+V` / `Cmd+V`), album ảnh dạng lưới, video, tin nhắn thoại và tập tin đính kèm.
- **Gợi ý Sticker theo ngữ cảnh** ngay khi gõ tiếng Việt có dấu / không dấu hoặc từ viết tắt (`xin chào`, `cảm ơn`, `tks`, `cố lên`, `mệt quá`...).
- Xem trước liên kết (link preview) và xem nhanh tài liệu trực tiếp trong ứng dụng (`PDF`, `DOC/DOCX`, `XLS/XLSX`, `PPT/PPTX`, `CSV/TSV`, `RTF`).

### Gọi thoại & Video 1-1
- Báo hiệu cuộc gọi VoIP và bắt tay mã hóa luồng âm thanh/hình ảnh trực tiếp bằng Rust (`cpal` + `opus`, tùy chọn `ffmpeg` cho gọi video H.264).

### Tiện ích mở rộng (Tùy chọn)
- **Kịch bản hội thoại (Rhai Scripting):** Tự động hóa phản hồi theo sự kiện hoặc từ khóa trên từng nhóm/hội thoại.
- **Trợ lý AI & Tóm tắt nhóm:** Kết nối tới endpoint OpenAI-compatible hoặc Ollama chạy cục bộ.
- **Cầu nối Telegram ↔ Zalo** và máy chủ **HTTP REST API / MCP (JSON-RPC 2.0)** cục bộ.

---

## Hướng dẫn biên dịch từ mã nguồn

### 1. Yêu cầu môi trường
- **Rust 1.75+** ([cài qua rustup.rs](https://rustup.rs/))
- **CMake (3.15+)**, `pkg-config` và trình biên dịch C/C++ (để biên dịch tĩnh `sqlite3` và `libopus`)
- **FFmpeg** trong `PATH` *(tùy chọn — chỉ cần nếu dùng gọi video hoặc trích xuất khung hình video)*

> **Lưu ý với CMake 4.0+:** Đặt biến môi trường `CMAKE_POLICY_VERSION_MINIMUM=3.5` khi biên dịch để tương thích với `opus-sys`.

### 2. Windows (PowerShell / MSVC)
Cài đặt *Visual Studio Build Tools* (chọn *Desktop development with C++* và *CMake tools for Windows*):

```powershell
git clone https://github.com/CgTuan210/zarust.git
cd zarust
$env:CMAKE_POLICY_VERSION_MINIMUM = "3.5"
cargo build --release --locked
```

File thực thi sau khi build: `target\release\ctdozlite.exe`

### 3. Linux (Ubuntu / Debian / Fedora / Arch)
Cài đặt các thư viện hệ thống cho âm thanh (`ALSA`) và cửa sổ (`X11` / `Wayland` / `GTK3`):

- **Ubuntu / Debian:**
  ```bash
  sudo apt update
  sudo apt install -y build-essential cmake pkg-config \
    libasound2-dev libxcb-render0-dev libxcb-shape0-dev \
    libxcb-xfixes0-dev libxkbcommon-dev libssl-dev \
    libgtk-3-dev zenity xclip wl-clipboard ffmpeg
  ```
- **Fedora:**
  ```bash
  sudo dnf install -y gcc gcc-c++ make cmake pkgconf-pkg-config \
    alsa-lib-devel libxcb-devel libxkbcommon-devel openssl-devel \
    gtk3-devel zenity xclip wl-clipboard ffmpeg
  ```
- **Arch Linux:**
  ```bash
  sudo pacman -S --needed base-devel cmake pkgconf alsa-lib \
    libxcb libxkbcommon openssl gtk3 zenity xclip wl-clipboard ffmpeg
  ```

Biên dịch:
```bash
git clone https://github.com/CgTuan210/zarust.git
cd zarust
CMAKE_POLICY_VERSION_MINIMUM=3.5 cargo build --release --locked
```

File thực thi sau khi build: `target/release/ctdozlite`

### 4. macOS (Apple Silicon & Intel)
```bash
xcode-select --install
brew install cmake ffmpeg
git clone https://github.com/CgTuan210/zarust.git
cd zarust
CMAKE_POLICY_VERSION_MINIMUM=3.5 cargo build --release --locked
```

File thực thi sau khi build: `target/release/ctdozlite`

---

## Cách chạy & Các tùy chọn CLI

```bash
# Chạy giao diện đồ họa Desktop (mặc định)
cargo run --release -- --gui

# Chạy giao diện Terminal (TUI) và đăng nhập bằng QR
cargo run --release -- --tui --qr

# Bật thêm bảng chẩn đoán mạng & sự kiện (Dev Inspector)
cargo run --release -- --gui --dev

# Chạy hồ sơ / thư mục dữ liệu riêng biệt
cargo run --release -- --gui --profile cong-viec
cargo run --release -- --gui --data-dir /duong/dan/thu-muc-du-lieu

# Đăng xuất và xóa phiên hiện tại
cargo run --release -- --logout
```

### Lệnh phát triển & Kiểm thử
```bash
# Kiểm tra quy chuẩn kiến trúc dự án
python3 scripts/check_invariants.py

# Chạy toàn bộ bộ kiểm thử tự động
CMAKE_POLICY_VERSION_MINIMUM=3.5 cargo test --locked
```

---

## Cấu trúc mã nguồn chính

| Đường dẫn | Vai trò |
| :--- | :--- |
| `src/main.rs`, `src/lib.rs` | Điểm vào CLI, khởi tạo thư mục dữ liệu và điều phối chế độ GUI / TUI |
| `src/gui.rs`, `src/gui/` | Giao diện desktop `egui` (`workspace`, `messages`, `modals`, `stickers`, `theme`, `worker`) |
| `src/gui/worker.rs` | Runtime `Tokio` nền xử lý mạng, đồng bộ, giải mã và ghi dữ liệu không chặn UI |
| `src/tui.rs`, `src/ui.rs` | Giao diện dòng lệnh trên terminal (`ratatui`) |
| `src/chat.rs`, `src/unread.rs` | Nghiệp vụ hội thoại, tin nhắn, nhóm, reaction, ghim, bộ đếm chưa đọc & tắt thông báo |
| `src/history_sync.rs`, `src/zproto_host.rs` | Đồng bộ lịch sử tin nhắn đám mây và giải mã gói đồng bộ từ điện thoại (Sync V1/V2) |
| `src/socket.rs`, `src/session.rs` | Kết nối WebSocket thời gian thực (`zpw_ws`), quản lý phiên, cookie và ký request |
| `src/param_cipher.rs`, `src/crypto.rs` | Mã hóa tham số `ParamCipher v2`, AES-CBC/GCM và tạo chữ ký `signkey` |
| `src/store.rs` | Cơ sở dữ liệu `SQLite` cục bộ theo từng tài khoản (`WAL`, giao dịch theo lô) |
| `src/call.rs`, `src/audio.rs`, `src/media.rs` | Báo hiệu VoIP, âm thanh `cpal`/`opus`, xử lý ảnh/video/tệp đính kèm |
| `docs/` | Tài liệu cài đặt, hướng dẫn sử dụng, kiến trúc hệ thống và tiêu chuẩn kỹ thuật |

---

## Tài liệu chi tiết

- [Cài đặt & gỡ cài đặt](docs/CAI_DAT.md)
- [Hướng dẫn sử dụng & phím tắt](docs/HUONG_DAN_SU_DUNG.md)
- [Danh mục tính năng & giới hạn hiện tại](docs/TINH_NANG.md)
- [Kiến trúc hệ thống](docs/KIEN_TRUC_HE_THONG.md)
- [Ghi chú giao thức](docs/GIAO_THUC_ZALO.md)
- [Khắc phục sự cố thường gặp](docs/KHAC_PHUC_SU_CO.md)

---

## Bảo mật & Quyền riêng tư

- Toàn bộ phiên đăng nhập và lịch sử tin nhắn chỉ lưu **cục bộ trên máy của bạn** (`session.json`, `accounts/`, `*.db`).
- **Tuyệt đối không chia sẻ hoặc đưa lên Git** các tệp `session.json`, `*.db`, cookie `zpw_sek` hay khóa `zpw_enk` (đã được chặn sẵn trong `.gitignore` và kiểm tra tự động qua `scripts/check_invariants.py`).
- Mặc định ứng dụng chỉ kết nối trực tiếp tới máy chủ Zalo (và GitHub Releases API khi kiểm tra bản mới). Dữ liệu hội thoại chỉ gửi tới bên thứ ba nếu bạn chủ động cấu hình và bật *Trợ lý AI* hoặc *Cầu nối Telegram*.

---

## Tuyên bố miễn trừ trách nhiệm & Giấy phép

> [!NOTE]
> Dự án được phát triển độc lập trên nền tảng mã nguồn mở [`ctdozlite`](https://github.com/ctdoteam1110/ctdozlite) nhằm mục đích **học tập, nghiên cứu kỹ thuật hệ thống với Rust và tương thích giao thức** (*Educational & Interoperability Research Purposes Only*).
>
> - Dự án **không** có bất kỳ mối liên hệ, hợp tác, tài trợ hay bảo trợ nào từ **VNG Corporation** hoặc **Zalo**. Tên gọi *Zalo* chỉ được dùng với ý nghĩa mô tả khả năng tương thích giao thức.
> - Dự án **hoàn toàn phi thương mại** (mã nguồn mở, không thu phí, không quảng cáo, không thu thập dữ liệu người dùng). Người dùng tự chịu trách nhiệm khi sử dụng phần mềm.

Phân phối dưới giấy phép kép **[MIT](LICENSE-MIT) OR [Apache-2.0](LICENSE-APACHE)**.
