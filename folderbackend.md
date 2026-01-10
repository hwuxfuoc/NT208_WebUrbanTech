### 1. models/ – "Bộ xương" của dữ liệu (BẮT BUỘC trong hầu hết dự án MongoDB)

**models/** là nơi định nghĩa **cấu trúc dữ liệu** (schema) và **mô hình** (model) mà bạn sẽ làm việc với MongoDB thông qua Mongoose (hoặc driver khác).

**Tại sao cần?**

- MongoDB là **schema-less** → bạn có thể lưu bất cứ thứ gì → rất dễ bị "rác" dữ liệu
- Khi dự án lớn lên (10–50+ collection), không ai nhớ hết trường nào bắt buộc, trường nào unique, ref với collection nào...
- Model giúp:
    - **Validate dữ liệu** trước khi lưu (required, minlength, enum, match email, v.v.)
    - Tự động thêm **timestamps** (createdAt, updatedAt)
    - Định nghĩa **relationship** (ref, populate)
    - Dễ **query** hơn (User.find({role: 'admin'}))
    - Dễ **test** và **document** cấu trúc dữ liệu

**Cấu trúc:**

```
models/
├── user.js
├── property.js
├── propertyImage.js
├── friend.js
├── friendRequest.js
├── conversation.js
├── message.js
├── favorite.js
└── aiConsultation.js
```

**Ví dụ User.js:**

```jsx
const mongoose = require('mongoose');

const userSchema = new mongoose.Schema({
    username: { type: String, required: true, unique: true },
    hashedPassword: { type: String, required: true },
    displayname: { type: String, required: true },
    email: { type: String, unique: true },
    avatarUrl: String,
    avatarId: String,
    bio: String,
    phone: String,
    updatedAt: { type: Date, default: Date.now },
    createdAt: { type: Date, default: Date.now },
});

userSchema.pre('save', async function() {
    this.updatedAt = Date.now();
});

module.exports = mongoose.model('User', userSchema);
```

**Kết luận**: Không có **models/** → dự án rất dễ thành "đống bãi rác dữ liệu".

### 2. api/ – "Trái tim" của ứng dụng (Nên có khi dự án trung bình → lớn)

**api/** (hoặc đôi khi gọi là **routes/** + **controllers/** hoặc **modules/**) là nơi tập trung **tất cả logic xử lý request HTTP**.

Hai cách tổ chức phổ biến hiện nay:

**Cách 1 – Tách domain (nhóm mình làm cách này)**

```
api/
├── auth/
│   ├── auth.routes.js
│   ├── auth.controller.js
│   └── auth.service.js      (tùy chọn – logic business)
├── property/
│   ├── property.routes.js
│   ├── property.controller.js
│   └── property.service.js
├── chat/
└── ai/

```

**Cách 2 – Gộp gọn hơn (không recommend cách này)**

```
api/
├── auth.js
├── property.js
├── chat.js
└── index.js     // gom tất cả router lại

```

**Ví dụ auth.routes.js + controller:**

```jsx
// backend/api/auth/auth.js
const express = require('express');
const router = express.Router();
const { register, login, getMe } = require('./authController');
const auth = require('../middleware/auth');

// @route   POST /api/auth/register
// @desc    Đăng ký user mới
// @access  Public
router.post('/register', register);

// @route   POST /api/auth/login
// @desc    Đăng nhập & trả về JWT token
// @access  Public
router.post('/login', login);

// @route   GET /api/auth/me
// @desc    Lấy thông tin user hiện tại
// @access  Private (cần token)
router.get('/me', auth, getMe);

module.exports = router;
```

```jsx
// backend/api/auth/authController.js
const bcrypt = require('bcryptjs');
const jwt = require('jsonwebtoken');
const { validationResult } = require('express-validator');
const User = require('../../models/user');

// Validate rules (sẽ dùng trong routes)
const registerValidator = [
  require('express-validator').body('username').notEmpty().withMessage('Username là bắt buộc'),
  require('express-validator').body('email').isEmail().withMessage('Email không hợp lệ'),
  require('express-validator').body('password').isLength({ min: 6 }).withMessage('Mật khẩu phải ít nhất 6 ký tự'),
  require('express-validator').body('displayname').notEmpty().withMessage('Tên hiển thị là bắt buộc'),
];

const loginValidator = [
  require('express-validator').body('email').isEmail().withMessage('Email không hợp lệ'),
  require('express-validator').body('password').notEmpty().withMessage('Mật khẩu là bắt buộc'),
];

// @desc    Đăng ký user mới
const register = [
  registerValidator,
  async (req, res) => {
    const errors = validationResult(req);
    if (!errors.isEmpty()) {
      return res.status(400).json({ errors: errors.array() });
    }

    const { username, email, password, displayname, phone } = req.body;

    try {
      // Kiểm tra username hoặc email đã tồn tại chưa
      let user = await User.findOne({ $or: [{ username }, { email }] });
      if (user) {
        return res.status(400).json({ message: 'Username hoặc email đã tồn tại' });
      }

      // Hash password
      const salt = await bcrypt.genSalt(10);
      const hashedPassword = await bcrypt.hash(password, salt);

      // Tạo user mới
      user = new User({
        username,
        email,
        hashedPassword,
        displayname,
        phone
      });

      await user.save();

      // Tạo JWT token
      const payload = { id: user.id };
      const token = jwt.sign(payload, process.env.JWT_SECRET, { expiresIn: '7d' });

      res.status(201).json({
        message: 'Đăng ký thành công',
        token,
        user: {
          id: user.id,
          username: user.username,
          displayname: user.displayname,
          email: user.email
        }
      });
    } catch (err) {
      console.error(err);
      res.status(500).json({ message: 'Lỗi server' });
    }
  }
];

// @desc    Đăng nhập
const login = [
  loginValidator,
  async (req, res) => {
    const errors = validationResult(req);
    if (!errors.isEmpty()) {
      return res.status(400).json({ errors: errors.array() });
    }

    const { email, password } = req.body;

    try {
      // Tìm user theo email
      const user = await User.findOne({ email });
      if (!user) {
        return res.status(400).json({ message: 'Email hoặc mật khẩu không đúng' });
      }

      // So sánh password
      const isMatch = await bcrypt.compare(password, user.hashedPassword);
      if (!isMatch) {
        return res.status(400).json({ message: 'Email hoặc mật khẩu không đúng' });
      }

      // Tạo token
      const payload = { id: user.id };
      const token = jwt.sign(payload, process.env.JWT_SECRET, { expiresIn: '7d' });

      res.json({
        message: 'Đăng nhập thành công',
        token,
        user: {
          id: user.id,
          username: user.username,
          displayname: user.displayname,
          email: user.email,
          avatarUrl: user.avatarUrl
        }
      });
    } catch (err) {
      console.error(err);
      res.status(500).json({ message: 'Lỗi server' });
    }
  }
];

// @desc    Lấy thông tin user hiện tại
const getMe = async (req, res) => {
  try {
    const user = await User.findById(req.user.id).select('-hashedPassword'); // Không trả password
    if (!user) {
      return res.status(404).json({ message: 'User không tồn tại' });
    }
    res.json({ user });
  } catch (err) {
    console.error(err);
    res.status(500).json({ message: 'Lỗi server' });
  }
};

module.exports = { register, login, getMe };
```

**Tại sao nên có api/ thay vì viết hết trong app.js?**

- Dự án 5 route thì ok, 50–200 route thì **app.js dài cả cây số**
- Dễ **phân quyền** (admin routes, agent routes, public routes...)
- Dễ **test** từng module riêng
- Teamwork tốt hơn (mỗi dev làm 1–2 domain)

### 3. config/ – "Hệ thống điều khiển" của toàn dự án (RẤT NÊN CÓ)

**config/** chứa các thiết lập hệ thống, không nên hard-code trong code.

**Lý do quan trọng nhất**:

- Cùng 1 codebase chạy ở **dev / staging / production** → cần thay đổi config mà **không sửa code**
- Dễ đưa vào **CI/CD**, Docker, Kubernetes
- Dễ **bảo mật** (Mongo URI, JWT secret, API key... để trong .env)

**Cấu trúc phổ biến:**

```
config/
├── db.js
├── cloudinary.js
├── firebase.js
├── passport_fb.js
└── passport_gg.js
```

**Ví dụ db.js:**

```jsx
require('dotenv').config();

const mongoose = require('mongoose');

const connectDB = async () => {
  try {
    await mongoose.connect(process.env.MONGO_URI);
    console.log('MongoDB connected successfully!');
  } catch (err) {
    console.error('MongoDB connection error:', err.message);
    process.exit(1);
  }
};

module.exports = connectDB;
```

### 4. logging/ – "Hộp đen" của hệ thống (Không bắt buộc cho MVP, nhưng rất chuyên nghiệp)

**logging/** quản lý log một cách có hệ thống thay vì `console.log` lung tung.

**Lợi ích lớn nhất**:

- Biết được **lỗi xảy ra khi nào, ở đâu, với dữ liệu gì**
- Dễ **debug production**
- Theo dõi hành vi người dùng (login, payment fail, API slow...)
- Dễ tích hợp **monitoring** (Sentry, Datadog, ELK stack...)

**Thư viện phổ biến 2025–2026**:

- **winston** (cổ điển, rất mạnh, nhiều transport)
- **pino** (nhanh nhất hiện nay, đặc biệt khi traffic cao)

**Ví dụ cấu trúc logging:**

```
logging/
└── enventLogger.js
```

```jsx
const fs = require('fs');
const path = require('path');

const EVENT_FILE_PATH = path.join(__dirname, '..', '..', 'suggestion', 'events.csv');

function logEvent(visitorId, productId, eventName) {
    const timestamp = Date.now()
    const dataRow = `${timestamp},${visitorId},${eventName},${productId}\n`
    
    const fileExists = fs.existsSync(EVENT_FILE_PATH)
    const header = "timestamp,visitorid,event,productID\n"

    try {
        if (!fileExists) {
            fs.writeFileSync(EVENT_FILE_PATH, header, { flag: 'w' })
        }
        fs.writeFileSync(EVENT_FILE_PATH, dataRow, { flag: 'a' })
    } catch (err) {
        console.error('❌ Error writing to events.csv:', err.message)
    }
}

module.exports = { logEvent };
```

File eventLogger này được lấy từ project khác chỉ tham khảo thôi

**Cách dùng:**

```jsx
const logger = require('../logging');

logger.info('User logged in', { userId: user._id, ip: req.ip });
logger.error('Payment failed', { error: err.message, transactionId });
```

### Tóm tắt nhanh – mức độ cần thiết cho dự án BĐS + Chat + AI

| Thư mục | Bắt buộc cho MVP? | Nên có khi scale? | Điểm chuyên nghiệp | Khó khăn nếu thiếu sau này |
| --- | --- | --- | --- | --- |
| models/ | ★★★★★ | ★★★★★ | ★★★★★ | Rất khó (dữ liệu loạn) |
| api/ | ★★★★ | ★★★★★ | ★★★★ | Khó maintain, refactor |
| config/ | ★★★★ | ★★★★★ | ★★★★ | Khó deploy, bảo mật |
| logging/ | ★★ | ★★★★ | ★★★★★ | Debug production cực khó |
