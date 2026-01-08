# NT208_WebUrbanTech
Description:...
Install: [Setup](Setup.md)
Database Graph: [Eraser](https://app.eraser.io/workspace/MibBxluYc33hORCIUomJ?origin=share)
MongoDB: [Mongo](mongodb+srv://hwuxfuoc19it_db_user:e76NQ2jZaEZEzvXN@clusterurbantech.97zwxie.mongodb.net?retryWrites=true&w=majority)
WebFrontend: [Web](http://localhost:3000)

# NT208_WebUrbanTech

**WebUrbanTech** — Một nền tảng bất động sản hiện đại, lấy cảm hứng từ các trang như [batdongsan.com.vn](http://batdongsan.com.vn/). Dự án này cho phép người dùng đăng ký/đăng nhập, đăng tin bán/thuê nhà đất với hình ảnh chi tiết, thông tin giá cả, vị trí; xem và tìm kiếm các tin đăng khác; kết bạn, chat trực tiếp với người bán; và tích hợp tư vấn bất động sản bằng AI trong tương lai.

Đây là dự án học tập và thực hành fullstack web development, được xây dựng từ con số 0 với mục tiêu áp dụng các công nghệ hiện đại trong thực tế.

### Tech Stack & Kiến Trúc Dự Án

Dự án sử dụng **MERN Stack** (MongoDB - Express - React - Node.js) — toàn bộ bằng JavaScript để dễ quản lý và học tập.

| Phần | Công nghệ chính | Mô tả ngắn gọn |
| --- | --- | --- |
| **Frontend** | React (với Create React App), JSX, Axios, React Router DOM, Material UI (MUI) | Giao diện người dùng: trang chủ, đăng tin, xem listing, profile, chat, đăng ký/đăng nhập. Chạy local trên port 3000. |
| **Backend** | Node.js, Express.js, Mongoose, JWT (auth), bcryptjs (hash password), cors, dotenv | Xử lý API (CRUD bất động sản, user, chat), authentication, kết nối database. Chạy local trên port 5000. |
| **Database** | MongoDB (MongoDB Atlas - free tier M0) | Lưu trữ dữ liệu user, property, message, conversation, favorite, v.v. theo schema đã thiết kế (ERD). |
| **Upload ảnh** | Cloudinary (free tier) hoặc tương tự (sẽ tích hợp sau) | Quản lý hình ảnh bất động sản. |
| **Deploy** | [Render.com](http://render.com/) (full app hoặc backend), Vercel/Netlify (frontend static) | Dự kiến deploy miễn phí khi hoàn thiện. |

Cấu trúc thư mục chính:

```
batdongsan-app/
├── backend/       # Server Node/Express + models/routes
├── frontend/      # React app
└── README.md
```

### Hướng Dẫn Cài Đặt & Chạy Local

Xem chi tiết tại: [Setup](Setup.md)

### Các Liên Kết Quan Trọng

| Nội dung | Link |
| --- | --- |
| **Database ER Diagram** | [Eraser.io Workspace](https://app.eraser.io/workspace/MibBxluYc33hORCIUomJ?origin=share) |
| **MongoDB Atlas** | [Mongo](mongodb+srv://hwuxfuoc19it_db_user:e76NQ2jZaEZEzvXN@clusterurbantech.97zwxie.mongodb.net?retryWrites=true&w=majority) |
| **Frontend (Local)** | [http://localhost:3000](http://localhost:3000/) |
| **Backend API (Local)** | [http://localhost:5000](http://localhost:5000/) |
| **Deploy Live** | (Sắp cập nhật khi deploy lên Render) |
