# Study With Me — Design Spec

**Date:** 2026-06-29  
**Status:** Draft  

## Overview

Trang web HTML tĩnh "Study With Me" — single file, mở bằng trình duyệt chạy ngay. Giao diện ấm áp cozy pastel, nền GIF pixel art lửa trại, hỗ trợ tập trung học tập với Pomodoro, nhạc nền YouTube, và todo list.

## Features

### 1. Pomodoro Timer (đồng hồ analog)
- **Chu kỳ:** 50 phút làm việc / 10 phút nghỉ
- **Hiển thị:** Mặt đồng hồ analog với kim đếm ngược chuyển động
  - Vòng tròn tiến trình (SVG/CSS) hiển thị % thời gian còn lại
  - Kim giây/phút chạy ngược
  - Chữ số hiển thị MM:SS ở giữa đồng hồ
- **Điều khiển:** Start/Pause, Reset
- **Hiệu ứng:** Rung nhẹ khi chuyển chế độ (focus → break), màu sắc thay đổi giữa chế độ làm việc (cam ấm) và nghỉ (xanh mint)

### 2. YouTube Audio Player
- **Input:** Ô nhập link YouTube + nút Load
- **Hoạt động:** Nhúng iframe YouTube ẩn (chỉ phát âm thanh), tự động phát
- **Hiển thị:** Tên bài/nhỏ + trạng thái đang phát
- **Hỗ trợ:** Link video thường, link live stream, link playlist

### 3. Todo List
- **Thêm:** Ô input + nút Add (hoặc Enter)
- **Hiển thị:** Danh sách các mục có checkbox
- **Tương tác:** Check để đánh dấu hoàn thành (gạch ngang), nút xóa từng mục
- **Lưu trữ:** LocalStorage — giữ lại danh sách khi tải lại trang

### 4. Visual Design
- **Nền:** `Lonely Camp Fire GIF.gif` — pixel art lửa trại, full screen, cover
- **Overlay:** Lớp phủ tối nhẹ (black 40%) để chữ dễ đọc
- **Card chính:** Glassmorphism (blur background, nền trong mờ, bo góc 20px)
- **Typography:** Font chữ tròn trịa thân thiện (Nunito/Quicksand từ Google Fonts)
- **Màu sắc:**
  - Working mode: cam ấm (#e07a5f)
  - Break mode: xanh mint (#81b29a)
  - Text: trắng kem (#fefae0)
  - Card background: rgba(255,255,255,0.12)

## Architecture

```
Single HTML file: index.html
├── <style> — Toàn bộ CSS
│   ├── CSS variables cho màu sắc
│   ├── Layout flexbox center
│   ├── Glassmorphism card
│   ├── Analog clock (CSS/SVG animation)
│   ├── Todo list styles
│   └── Responsive adjustments
├── <body> — Cấu trúc
│   ├── Background GIF (CSS background-image)
│   ├── Main card container
│   │   ├── Title "☕ Study With Me"
│   │   ├── Pomodoro section (clock + controls)
│   │   ├── YouTube section (input + player)
│   │   └── Todo section (list + input)
│   └── Hidden YouTube iframe
└── <script> — Toàn bộ JavaScript
    ├── Pomodoro logic (timer, clock animation)
    ├── YouTube API (iframe embed/control)
    ├── Todo CRUD + LocalStorage
    └── Audio notification (chuông báo hết giờ)
```

## Technical Decisions

| Decision | Choice | Reason |
|----------|--------|--------|
| Analog clock | SVG circle + CSS transform | Mượt, nhẹ, không cần canvas |
| YouTube embed | iframe API | Đơn giản, đủ dùng, không cần API key |
| Background | CSS `background-image` với `background-size: cover` | Đơn giản nhất |
| State persistence | `localStorage` | Không cần backend |
| Font | Google Fonts CDN (Nunito) | Tải 1 lần, cache tốt |
| Âm báo | Web Audio API tone đơn giản | Không cần file âm thanh ngoài |

## Error & Edge Cases

- **YouTube link không hợp lệ:** Hiển thị message lỗi nhẹ, không crash
- **Chưa nhập link đã bấm Load:** Focus vào input, không làm gì
- **Todo rỗng:** Hiển thị placeholder "Chưa có việc nào..."
- **LocalStorage đầy/lỗi:** Fail silently, todo vẫn dùng được trong phiên
- **GIF không tải được:** Fallback màu nền gradient
- **Chuyển tab khi đang focus:** Timer vẫn chạy đúng (dùng Date timestamp)

## File Structure

```
studywme/
├── index.html                  # File chính duy nhất
├── Lonely Camp Fire GIF.gif   # Ảnh nền
└── docs/superpowers/specs/
    └── 2026-06-29-study-with-me-design.md
```
