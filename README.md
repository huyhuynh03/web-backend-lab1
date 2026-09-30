# Web Backend Lab 1 - Node.js, MySQL, MongoDB

Sinh viên: **Nguyễn Phan Thảo Nguyên** - MSSV **26110906** - Lớp **26DTHC2**

```
web-backend-lab1/
├── lab1-backend/
│   ├── server.js          # Exercise 1 - Express server (+ endpoint /api/greeting)
│   ├── test-endpoints.js  # Exercise 1 - kiểm tra nhanh các endpoint
│   ├── package.json       # Exercise 1 - npm project (express + dotenv)
│   └── .env.example       # Exercise 1 - mẫu biến môi trường
├── lab2-mysql/
│   ├── 01_schema.sql      # Exercise 2 - DDL (database + 4 bảng, FK)
│   ├── 02_seed_data.sql   # Exercise 2 - DML (INSERT users/products; Q1 orders/order_items)
│   └── 03_queries.sql     # Exercise 2 - DQL (Q2 filter, Q3 JOIN, Q4 GROUP BY/SUM)
├── lab3-mongodb/
│   ├── 01_mongosh_basics.js    # Exercise 3 - shop_db/orders, insertMany, find, updateOne
│   └── 02_mongosh_extended.js  # Exercise 3 mở rộng - Q1->Q4 (array query, $push/$inc, aggregation)
├── package.json           # scripts ở root: npm start, npm run test:api
└── .gitignore             # loại trừ node_modules/.env
```

## 0. Chuẩn bị (chạy 1 lần)

```cmd
cd d:\school_exercise\2_hk1\web\web-backend-lab1\lab1-backend
npm install
copy ..\.env NUL 2>NUL & type ..\.env
git init & git remote add origin https://github.com/huyhuynh03/web-backend-lab1.git
```

Nếu đã có repo, thay `git init` bằng:

```cmd
git clone https://github.com/huyhuynh03/web-backend-lab1.git .
```

## 1. Exercise 1 - Node.js + Express (cổng 5000)

```cmd
cd d:\school_exercise\2_hk1\web\web-backend-lab1\lab1-backend
npm start
```

Mở trình duyệt:

- `http://localhost:5000/` -> `{ message: "Welcome to Web Backend & Database Lab 1 API", ... }`
- `http://localhost:5000/api/health` -> `{ status: "OK", uptime: ... }`
- `http://localhost:5000/api/greeting` (yêu cầu mở rộng) -> thông tin sinh viên:

```json
{
  "success": true,
  "message": "Hello, this is my information",
  "student": {
    "fullName": "Nguyen Phan Thao Nguyen",
    "studentId": "26110906",
    "class": "26DTHC2"
  },
  "timestamp": "2026-09-11T08:51:41.247Z"
}
```

Kiểm tra bằng script có sẵn (chạy ở terminal khác khi server đang chạy):

```cmd
node test-endpoints.js
```

Đổi cổng qua biến môi trường nếu cổng 5000 bận:

```cmd
set PORT=5001 && node server.js
```

## 2. Exercise 2 - MySQL (Workbench) - cơ sở dữ liệu `ecommerce_db`

1. Kết nối MySQL Workbench -> mở file `lab2-mysql/`.
2. Chạy **theo thứ tự**: `01_schema.sql` -> `02_seed_data.sql` -> `03_queries.sql`.
3. Liên kết bảng (ràng buộc khóa ngoại):

| Bảng con       | Khóa ngoại | Tham chiếu tới |
|----------------|------------|----------------|
| `orders.user_id`       | `fk_orders_user`        | `users(id)`    |
| `order_items.order_id` | `fk_order_items_order`  | `orders(id)`   |
| `order_items.product_id` | `fk_order_items_product` | `products(id)` |

4. Đối chiếu yêu cầu mở rộng với file SQL:

| Yêu cầu | Vị trí trong file |
|---------|-------------------|
| Q1 - INSERT 3+ `orders` + chi tiết `order_items` | `02_seed_data.sql` (mục 3-4) |
| Q2 - products giá 100.000-1.000.000, ORDER BY price DESC | `03_queries.sql` (mục Q2) |
| Q3 - INNER JOIN 4 bảng: Order ID, Customer Name, Product Name, Quantity, Unit Price, Order Status | `03_queries.sql` (mục Q3, bản Q3b/Q3c có cột tổng tiền/bản completed) |
| Q4a - tổng doanh thu completed | `03_queries.sql` (mục Q4a) |
| Q4b - số đơn mỗi user (GROUP BY user_id) | `03_queries.sql` (mục Q4b; Q4c/Q4d là thống kê thêm) |

Mẫu dữ liệu `orders` (tự động kiểm tra trong `02_seed_data.sql`: `total_amount` = tổng `quantity*price`):

| id | user_id | total_amount | status    |
|----|---------|--------------|-----------|
| 1  | 1       | 27,400,000   | completed |
| 2  | 2       | 18,000,000   | pending   |
| 3  | 3       | 8,500,000    | completed |

## 3. Exercise 3 - MongoDB (Compass / mongosh) - cơ sở dữ liệu `shop_db`

Kết nối: `mongodb://localhost:27017` (hoặc chuỗi Atlas của bạn).

### 3a. Phần cơ bản (file `01_mongosh_basics.js`)

```cmd
mongosh "mongodb://localhost:27017" lab3-mongodb\01_mongosh_basics.js
```

Hoặc mở MongoDB Compass -> kết nối -> mở Mongosh bên dưới -> copy từng khối lệnh trong file.

Nội dung file gồm đúng yêu cầu đề:

1. `use shop_db;` (trong IDE dùng `db = db.getSiblingDB('shop_db');` tương đương),
2. `db.orders.insertMany([...])` chèn 2 đơn `ORD-2026-001/002` có mảng `items` (sub-document),
3. `db.orders.find({ status: "completed" }).pretty()`,
4. `db.orders.updateOne({ order_code: "ORD-2026-002" }, { $set: { status: "processing" } })`.

### 3b. Phần mở rộng (file `02_mongosh_extended.js`)

```cmd
mongosh "mongodb://localhost:27017" lab3-mongodb\02_mongosh_extended.js
```

| Yêu cầu | Câu lệnh trong file |
|---------|---------------------|
| Q1 - thêm `ORD-2026-003` (completed, 3 sản phẩm trong `items`) và `ORD-2026-004` (cancelled) | `insertMany` đầu file |
| Q2 - `total_amount >= 5_000_000` AND `status: "completed"` | `find({ total_amount: { $gte: 5000000 }, status: "completed" })` |
| Q2 - đơn chứa `"Logitech MX Master 3S Mouse"` trong `items` | `find({ "items.product_name": "..." })` (+ `$elemMatch` chỉ hiện phần tử khớp) |
| Q3 - thêm `{ product_name: "XL Gaming Mouse Pad", quantity: 1, price: 200000 }` vào `ORD-2026-002` ($push) và cộng 200.000 vào `total_amount` ($inc) | `updateOne({ order_code: "ORD-2026-002" }, { $push: ..., $inc: ... })` |
| Q4 - tổng doanh thu completed ($match + $group) | `aggregate([{ $match: { status: "completed" } }, { $group: { _id: null, total_revenue: { $sum: "$total_amount" } } }])` |
| Q4 - đếm đơn theo `status` | `aggregate([{ $group: { _id: "$status", order_count: { $sum: 1 } } }])` |

## 4. Đẩy bài lên GitHub

```cmd
cd d:\school_exercise\2_hk1\web\web-backend-lab1
git add .
git commit -m "Lab 1: Express API + MySQL schema/queries + MongoDB scripts"
git branch -M main
git push -u origin main
```
