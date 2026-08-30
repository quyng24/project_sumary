# 🚀 Svelte Showcase Portfolio

Một hệ sinh thái giao diện thử nghiệm được xây dựng trên nền tảng **Svelte 5 (Runes)**. Dự án tập trung vào việc tối ưu hóa hiệu suất (GPU Acceleration), trải nghiệm người dùng mượt mà và thiết kế Minimalist Modern.

![Svelte 5](https://img.shields.io/badge/Svelte-5.0_Runes-ff3e00?logo=svelte&logoColor=white)
![SvelteKit](https://img.shields.io/badge/SvelteKit-Latest-000?logo=svelte&logoColor=ff3e00)
![Vite](https://img.shields.io/badge/Vite-Latest-646CFF?logo=vite&logoColor=white)

---

## ✨ Điểm nổi bật

- **Svelte 5 Magic:** Sử dụng hoàn toàn hệ thống Runes (`$state`, `$derived`, `$effect`) để quản lý reactivity hiệu quả.
- **High Performance:**
  - Background sao băng & cực quang được tối ưu hóa 100% bằng GPU.
  - Zero-lag Hydration: Khắc phục triệt để lỗi treo trang khi chuyển đổi giữa SSR và Client.
- **Premium UX:** Thiết kế theo phong cách Minimalist Tech, tập trung vào Typography và Interactive List-Flow thay vì các thẻ card truyền thống.
- **Responsive Design:** Hiển thị hoàn hảo từ màn hình Desktop 4K đến các thiết bị di động nhỏ nhất.

---

## 🛠 Danh mục dự án (Lab Experiments)

Dự án này bao gồm một tập hợp các công cụ và trò chơi nhỏ:

1.  **🕰 Flip Clock:** Đồng hồ cơ học với hiệu ứng lật trang 3D chân thực (Mechanical Flip).
2.  **📊 Excel Web:** Trình soạn thảo bảng tính mini với engine tính toán công thức (`SUM`, `AVG`, `RANGE`).
3.  **🕹 Game Suite:**
    - **Tetris:** Xử lý logic mảng 2D và va chạm vật lý.
    - **Tictactoe:** Giao diện Neon Cyberpunk.
    - **Platformer:** Game đi ải 2D tập trung vào Game Loop và Camera bám đuổi.
4.  **🎄 Christmas Tree:** Diễn họa cây thông Noel bằng mã nhị phân kết hợp hiệu ứng tuyết rơi GPU.
5.  **💖 Heart Particle:** Hệ thống hạt (Particles) mô phỏng vật lý gia tốc trên Canvas 2D.
6.  **🪐 Animate Planet:** Mô phỏng quỹ đạo hành tinh bằng SVG Animation.

---

## ⚙️ Cài đặt & Khởi chạy

Đảm bảo bạn đã cài đặt [Node.js](https://nodejs.org/) (phiên bản 18 trở lên).

1. **Clone dự án:**
   ```bash
   git clone https://github.com/quyng24/projects_sumary.git
   cd projects_sumary
   npm install
   npm run dev
   npm run build
   ```

## 📁 Cấu trúc thư mục

    src/
    ├── lib/
    │    ├── assets/       # Hình ảnh, âm thanh (.mp3, .svg)
    │    └── components/   # Các component dùng chung (BackHome, Nav...)
    ├── routes/
    │    ├── clock/        # Dự án Flip Clock
    │    ├── excel/        # Dự án Spreadsheet
    │    ├── tetris/       # Dự án Game Tetris
    │    ├── christmastree/# Dự án Cây thông nhị phân
    │    └── ...           # Các trang con khác
    └── app.html           # File template HTML gốc
