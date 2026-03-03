# 宠物商店管理系统文档

## 一、核心架构

### 后端
- **技术栈**: Node.js + Express + SQLite3 + Socket.io
- **API设计**: RESTful风格
- **实时通信**: WebSocket

### 前端
- **技术栈**: React + TypeScript + Tailwind CSS
- **API对接**: Axios (封装在 lib/api.js)
- **WebSocket**: socket.io-client

### 数据库
- **数据库**: SQLite3
- **ORM**: Sequelize

## 二、后端核心功能

### 1. 商品管理模块

#### API接口

| 方法 | 路径 | 描述 |
|------|------|------|
| GET | /api/products | 获取商品列表 |
| GET | /api/products/:id | 获取单个商品详情 |
| POST | /api/products | 添加商品 |
| PUT | /api/products/:id | 修改商品 |
| DELETE | /api/products/:id | 删除商品 |

#### 查询参数 (GET /api/products)
- `category`: 分类筛选
- `status`: 状态筛选 (active/inactive)
- `page`: 页码 (默认1)
- `limit`: 每页数量 (默认20)

#### WebSocket推送事件
- `product:created`: 商品创建
- `product:updated`: 商品更新
- `product:deleted`: 商品删除

#### 测试用例 (curl)

```bash
# 获取商品列表
curl http://localhost:3001/api/products

# 添加商品
curl -X POST http://localhost:3001/api/products \
  -H "Content-Type: application/json" \
  -d '{
    "name": "新商品测试",
    "price": 99.99,
    "category": "food",
    "description": "商品描述",
    "brand": "测试品牌"
  }'

# 更新商品
curl -X PUT http://localhost:3001/api/products/1 \
  -H "Content-Type: application/json" \
  -d '{"price": 129.99}'

# 删除商品
curl -X DELETE http://localhost:3001/api/products/1
```

### 2. 用户数据监控模块

#### API接口

| 方法 | 路径 | 描述 |
|------|------|------|
| GET | /api/admin/users | 获取所有用户列表 |
| GET | /api/admin/users/:id | 获取单个用户详情 |
| GET | /api/admin/users/:id/logs | 获取用户行为日志 |

#### 查询参数
- `status`: 用户状态筛选 (active/muted/banned)
- `page`: 页码
- `limit`: 每页数量

#### 用户行为日志 action 类型
- `login`: 登录
- `logout`: 登出
- `view`: 查看商品
- `add_to_cart`: 添加到购物车
- `purchase`: 购买

### 3. 账号管控模块

#### API接口

| 方法 | 路径 | 描述 |
|------|------|------|
| PUT | /api/admin/users/:id/mute | 禁言用户 |
| PUT | /api/admin/users/:id/unmute | 解除禁言 |
| PUT | /api/admin/users/:id/ban | 封号用户 |
| PUT | /api/admin/users/:id/unban | 解封用户 |
| DELETE | /api/admin/users/:id | 删除用户 |

#### WebSocket推送事件
- `user:statusChanged`: 用户状态变更
- `user:deleted`: 用户删除
- `user:action`: 用户行为

#### 测试用例 (curl)

```bash
# 获取用户列表
curl http://localhost:3001/api/admin/users

# 禁言用户
curl -X PUT http://localhost:3001/api/admin/users/1/mute

# 解除禁言
curl -X PUT http://localhost:3001/api/admin/users/1/unmute

# 封号用户
curl -X PUT http://localhost:3001/api/admin/users/1/ban

# 解封用户
curl -X PUT http://localhost:3001/api/admin/users/1/unban

# 删除用户
curl -X DELETE http://localhost:3001/api/admin/users/1

# 获取用户行为日志
curl http://localhost:3001/api/admin/users/1/logs
```

## 三、开发要求

### 后端代码结构

```
server/
├── src/
│   ├── config/
│   │   └── config.js          # 配置文件
│   ├── database.js             # 数据库模型
│   ├── index.js                # 服务器入口
│   └── routes/
│       ├── paymentRoutes.js    # 支付相关路由
│       ├── userRoutes.js       # 用户相关路由
│       ├── adminRoutes.js      # 管理员路由
│       ├── productRoutes.js    # 商品管理路由 (新增)
│       └── adminUserRoutes.js  # 用户管理路由 (新增)
```

### 前端API使用

#### 商品管理
```javascript
import { productAPI } from '../lib/api';

// 获取商品列表
const products = await productAPI.getProducts({ category: 'food', page: 1 });

// 添加商品
await productAPI.addProduct({
  name: '商品名称',
  price: 99.99,
  category: 'food'
});

// 更新商品
await productAPI.updateProduct(id, { price: 129.99 });

// 删除商品
await productAPI.deleteProduct(id);
```

#### 用户管理
```javascript
import { adminUserAPI } from '../lib/api';

// 获取用户列表
const users = await adminUserAPI.getUsers({ status: 'active' });

// 禁言用户
await adminUserAPI.muteUser(userId);

// 封号用户
await adminUserAPI.banUser(userId);

// 获取用户日志
const logs = await adminUserAPI.getUserLogs(userId);
```

#### WebSocket监听
```javascript
import { io } from 'socket.io-client';

const socket = io('http://localhost:3001');

// 监听商品更新
socket.on('product:created', (product) => {
  console.log('新商品:', product);
});

socket.on('product:updated', (product) => {
  console.log('商品更新:', product);
});

// 监听用户状态变更
socket.on('user:statusChanged', (data) => {
  console.log('用户状态变更:', data);
});

socket.on('user:action', (data) => {
  console.log('用户行为:', data);
});
```

## 四、部署步骤

### 1. 安装依赖

```bash
# 安装前端依赖
cd pet-store
pnpm install

# 安装后端依赖
cd server
pnpm install
```

### 2. 启动服务

```bash
# 启动后端服务 (终端1)
cd server
pnpm run dev

# 启动前端服务 (终端2)
cd pet-store
pnpm run dev
```

### 3. 数据库初始化

数据库会在启动时自动创建和同步，无需手动初始化。

数据库文件位置: `server/pet_store.db`

### 4. 访问地址

- **前端**: http://localhost:5173/
- **后端API**: http://localhost:3001/
- **后端健康检查**: http://localhost:3001/health

## 五、测试用例

### 前端测试步骤

1. **商品添加测试**
   - 打开管理后台
   - 点击"添加商品"
   - 填写商品信息
   - 提交后查看是否实时显示

2. **用户禁言测试**
   - 在用户列表中选择一个用户
   - 点击"禁言"按钮
   - 尝试用该用户登录或发布内容
   - 验证功能受限

3. **WebSocket实时测试**
   - 打开两个浏览器窗口
   - 窗口A: 打开管理后台用户列表
   - 窗口B: 登录一个用户
   - 在窗口A中观察用户行为日志是否实时更新

### curl命令测试

详见上文各模块的测试用例。

## 六、核心逻辑说明

### 用户状态处理

用户状态有三种:
- `active`: 正常状态，可以正常使用所有功能
- `muted`: 禁言状态，无法发布内容/评论
- `banned`: 封号状态，无法登录

### 数据库模型

#### User表
- id: 用户ID
- name: 用户名
- email: 邮箱
- password: 密码 (加密存储)
- status: 用户状态
- isAdmin: 是否管理员
- createdAt: 创建时间

#### Product表
- id: 商品ID
- name: 商品名称
- description: 描述
- price: 价格
- image: 图片URL
- category: 分类
- brand: 品牌
- tags: 标签 (JSON)
- stock: 库存
- status: 上架状态
- createdAt: 创建时间

#### UserLog表
- id: 日志ID
- userId: 用户ID
- action: 行为类型
- details: 详细信息 (JSON)
- ip: IP地址
- userAgent: 浏览器信息
- createdAt: 创建时间

## 七、注意事项

1. **安全性**: 生产环境建议使用JWT进行API鉴权
2. **密码安全**: 用户密码应使用bcrypt等加密
3. **CORS配置**: 生产环境应配置具体的域名白名单
4. **WebSocket**: 当前WebSocket允许所有来源连接，生产环境应限制
5. **数据库**: SQLite适合开发和小型应用，生产环境建议使用MySQL/PostgreSQL
