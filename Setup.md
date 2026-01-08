### 1. Chuẩn Bị Môi Trường Chung (Cần Cho Cả Frontend Và Backend)

- **Bước 1: Cài Node.js**
    - Truy cập trang chính thức: [https://nodejs.org](https://nodejs.org/)
    - Tải phiên bản LTS (Long Term Support, ổn định nhất, hiện tại là 20.x hoặc mới hơn vào 2026).
    - Cài đặt như phần mềm bình thường (next next finish).
    - Kiểm tra cài thành công: Mở terminal, chạy lệnh `node -v` và `npm -v`. Nếu thấy version (ví dụ: v20.10.0 và 10.2.3) là OK.
    - Nếu lỗi, restart máy và thử lại.
- **Bước 2: Cài Git**
    - Truy cập: https://git-scm.com/downloads
    - Tải và cài đặt (next next).
    - Kiểm tra: `git --version`.
- **Bước 3: Cài VS Code**
    - Truy cập: [https://code.visualstudio.com](https://code.visualstudio.com/)
    - Tải và cài.
    - Mở VS Code, cài extension: "ESLint", "Prettier", "MongoDB for VS Code".

### 2. Chuẩn Bị MongoDB (Database)

- Dùng **MongoDB Atlas** 
- **Bước 1: Đăng ký tài khoản MongoDB Atlas**
    - Truy cập: https://www.mongodb.com/atlas/database
    - Click "Sign Up" → Đăng ký bằng email hoặc Google account (miễn phí).
- **Bước 2: Tạo Cluster (Database Server) miễn phí**
    - Sau đăng nhập, click "Build a Database" hoặc "Create" ở dashboard.
    - Chọn **Shared** (miễn phí) → Click "Create" cho tier M0 (free forever, 512MB storage đủ cho học).
    - Chọn Provider: AWS (mặc định).
    - Chọn Region gần Việt Nam nhất: "Asia Pacific - Singapore" (ap-southeast-1) để tốc độ nhanh.
    - Đặt tên Cluster: `ClusterUrbanTech`
    - Click "Create Cluster" → Chờ 2-5 phút.
- **Bước 3: Tạo User Database (để connect từ code)**
    - Sau khi cluster sẵn sàng, vào tab "Database Access" (bên trái).
    - Click "Add New Database User".
    - Username: `hwuxfuoc19it_db_user`
    - Password:  `e76NQ2jZaEZEzvXN`
    - Role: "Atlas Admin" (để full quyền).
    - Click "Add User".
- **Bước 4: Cho phép kết nối từ mọi nơi (Network Access)**
    - Vào tab "Network Access".
    - Click "Add IP Address".
    - Chọn "Allow Access from Anywhere" (0.0.0.0/0) → Confirm. (Dành cho dev, sau này hạn chế nếu cần security).
- **Bước 5: Lấy Connection String (chuỗi kết nối)**
    - Vào tab "Database" → Click "Connect" bên cạnh cluster.
    - Chọn "Drivers" (hoặc "Connect your application").
    - Chọn Node.js (version 4.0 or later).
    - **Connection String**: `mongodb+srv://hwuxfuoc19it_db_user:e76NQ2jZaEZEzvXN@clusterurbantech.97zwxie.mongodb.net?retryWrites=true&w=majority`
- **Bước 6: Test kết nối (tùy chọn, dùng VS Code hoặc Compass)**
    - Cài MongoDB Compass (từ https://www.mongodb.com/products/tools/compass).
    - Paste connection string vào Compass → Connect → Nếu thấy database là OK.

### 3. Chuẩn Bị Backend (Node.js + Express)

- Backend sẽ xử lý API (register, login, CRUD property, chat), connect DB.
- **Bước 1: Tạo thư mục dự án**
    - Mở terminal, chạy:
        
        ```
        mkdir weburbantech
        cd weburbantech
        mkdir backend
        cd backend
        ```
        
- **Bước 2: Khởi tạo project Node.js**
    - Chạy: `npm init -y` (tạo file package.json).
- **Bước 3: Cài packages cần thiết**
    - Cài core: `npm install express mongoose dotenv cors bcryptjs jsonwebtoken`
        - Express: Framework backend.
        - Mongoose: Connect và query MongoDB dễ dàng.
        - Dotenv: Lưu secret (như connection string).
        - Cors: Cho frontend connect backend.
        - Bcryptjs: Hash password.
        - Jsonwebtoken: JWT cho auth.
    - Cài dev tool: `npm install nodemon --save-dev` (tự restart server khi code thay đổi).
- **Bước 4: Tạo file cấu hình cơ bản**
    - Tạo file `.env` (lưu secret):
        
        ```
        MONGO_URI=mongodb+srv://hwuxfuoc19it_db_user:e76NQ2jZaEZEzvXN@clusterurbantech.97zwxie.mongodb.net?retryWrites=true&w=majority  
        PORT=5000
        JWT_SECRET=mysecretkey 
        ```
        
    - Tạo file `server.js` (entry point):
        
        ```jsx
        const express = require('express');
        const mongoose = require('mongoose');
        const dotenv = require('dotenv');
        const cors = require('cors');
        
        dotenv.config();
        
        const app = express();
        app.use(express.json());  
        app.use(cors()); 
        
        // Connect MongoDB
        mongoose.connect(process.env.MONGO_URI)
          .then(() => console.log('MongoDB connected'))
          .catch(err => console.log(err));
        
        // Route test
        app.get('/', (req, res) => res.send('Backend running!'));
        
        const PORT = process.env.PORT || 5000;
        app.listen(PORT, () => console.log(`Server on port ${PORT}`));
        ```
        
- **Bước 5: Chạy test backend**
    - Chỉnh package.json: Thêm vào "scripts": `"start": "nodemon server.js"`
    - Chạy: `npm start`
    - Mở browser: http://localhost:5000/ → Nếu thấy "Backend running!" là OK.
    - Nếu lỗi connect DB, kiểm tra MONGO_URI trong .env.

### 4. Chuẩn Bị Frontend (React với JSX)

- Frontend sẽ là giao diện (login, đăng tin, chat), gọi API từ backend.
- **Bước 1: Tạo React app**
    - Từ thư mục chính `weburbantech` , chạy:
        
        ```
        npx create-react-app frontend
        cd frontend
        ```
        
- **Bước 2: Cài packages cần thiết**
    - `npm install axios react-router-dom` (Axios: Gọi API; Router: Navigation).
    - Nếu cần UI đẹp: `npm install @mui/material @emotion/react @emotion/styled`.
- **Bước 3: Cấu hình cơ bản**
    - Mở `src/App.js`, chỉnh thành:
        
        ```jsx
        import logo from './logo.svg';
        import './App.css';
        
        function App() {
          return (
            <div className="App">
              <header className="App-header">
                <img src={logo} className="App-logo" alt="logo" />
                <p>
                  Frontend
                </p>
                <a
                  className="App-link"
                  href="https://github.com/hwuxfuoc/NT208_WebUrbanTech"
                  target="_blank"
                  rel="noopener noreferrer"
                >
                  GitHub
                </a>
              </header>
            </div>
          );
        }
        
        export default App;
        ```
        
- **Bước 4: Chạy test frontend**
    - Chạy: `npm start`
    - Mở browser: http://localhost:3000/ → Thấy "Xin chào từ Frontend!" là OK.
    - Để connect backend: Trong code, dùng axios.get('http://localhost:5000/') để test.
