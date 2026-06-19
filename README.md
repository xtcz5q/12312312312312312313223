# 技能五子棋后端系统

## 📖 项目概述

这是一个完整的技能五子棋游戏后端系统，提供以下核心功能：

- **用户认证系统**：注册、登录、JWT令牌认证
- **数据同步功能**：实时同步游戏数据、用户设置、游戏状态
- **玩家信息管理**：账户管理、积分系统、在线状态
- **数据持久化**：MongoDB数据库存储所有游戏数据
- **RESTful API**：完整的HTTP API接口
- **WebSocket实时通信**：实时游戏对战、消息推送
- **安全保障**：数据加密、防篡改、速率限制

## 🚀 快速开始

### 1. 环境要求

- Node.js >= 14.0.0
- MongoDB >= 4.0
- npm 或 yarn

### 2. 安装依赖

```bash
cd gomoku-backend
npm install
```

### 3. 配置环境变量

编辑 `.env` 文件，配置以下参数：

```env
# 服务器配置
PORT=3000
NODE_ENV=development

# MongoDB 数据库连接
MONGODB_URI=mongodb://localhost:27017/gomoku_game

# JWT 密钥（生产环境请修改）
JWT_SECRET=your-super-secret-jwt-key-change-this-in-production

# CORS 配置（允许的前端域名）
CORS_ORIGIN=http://localhost:3000,http://127.0.0.1:3000
```

### 4. 启动MongoDB

确保MongoDB服务正在运行：

```bash
# Windows
mongod

# Linux/Mac
sudo systemctl start mongodb
# 或
mongod
```

### 5. 启动服务器

```bash
# 开发模式（自动重启）
npm run dev

# 生产模式
npm start
```

服务器启动后，访问：
- API地址：http://localhost:3000/api
- WebSocket：ws://localhost:3000
- 健康检查：http://localhost:3000/health

## 📡 API 接口文档

### 认证接口 `/api/auth`

#### 注册
```http
POST /api/auth/register
Content-Type: application/json

{
  "phone": "13800138000",
  "password": "123456",
  "nickname": "玩家小明",
  "avatar": "😊"
}
```

**响应：**
```json
{
  "success": true,
  "message": "注册成功",
  "data": {
    "user": { ... },
    "token": "eyJhbGciOiJIUzI1NiIs..."
  }
}
```

#### 登录
```http
POST /api/auth/login
Content-Type: application/json

{
  "phone": "13800138000",
  "password": "123456"
}
```

#### 游客登录
```http
POST /api/auth/guest
```

#### 获取当前用户信息
```http
GET /api/auth/me
Authorization: Bearer {token}
```

#### 更新用户信息
```http
PUT /api/auth/me
Authorization: Bearer {token}
Content-Type: application/json

{
  "nickname": "新昵称",
  "avatar": "🎮"
}
```

#### 登出
```http
POST /api/auth/logout
Authorization: Bearer {token}
```

### 玩家接口 `/api/players`

#### 获取排行榜
```http
GET /api/players/leaderboard?limit=100&type=all
```

参数说明：
- `limit`: 返回数量（默认100）
- `type`: 玩家类型（all/real/ai）

#### 获取玩家详情
```http
GET /api/players/{phone}
```

#### 获取所有玩家（管理员）
```http
GET /api/players/all?includeAI=true
Authorization: Bearer {token}
```

#### 发放积分（管理员）
```http
POST /api/players/points/add
Authorization: Bearer {token}
Content-Type: application/json

{
  "phone": "13800138000",
  "amount": 100
}
```

#### 扣除积分（管理员）
```http
POST /api/players/points/reduce
Authorization: Bearer {token}
Content-Type: application/json

{
  "phone": "13800138000",
  "amount": 50
}
```

### 游戏接口 `/api/games`

#### 创建游戏
```http
POST /api/games
Authorization: Bearer {token}
Content-Type: application/json

{
  "players": [
    { "userId": "...", "color": "black" },
    { "userId": "...", "color": "white" }
  ],
  "gameMode": "pvp",
  "difficulty": "medium"
}
```

#### 更新游戏状态
```http
PUT /api/games/{gameId}
Authorization: Bearer {token}
Content-Type: application/json

{
  "moveHistory": [...],
  "winner": "...",
  "result": "win"
}
```

#### 获取游戏详情
```http
GET /api/games/{gameId}
```

#### 获取我的游戏记录
```http
GET /api/games/my?page=1&limit=20
Authorization: Bearer {token}
```

### 消息接口 `/api/messages`

#### 获取消息列表
```http
GET /api/messages?page=1&limit=50
```

#### 发送消息
```http
POST /api/messages
Authorization: Bearer {token}
Content-Type: application/json

{
  "content": "大家好！",
  "messageType": "text"
}
```

#### 添加评论
```http
POST /api/messages/{messageId}/comment
Authorization: Bearer {token}
Content-Type: application/json

{
  "content": "很棒！"
}
```

#### 点赞/取消点赞
```http
POST /api/messages/{messageId}/like
Authorization: Bearer {token}
```

### 数据同步接口 `/api/sync`

#### 同步数据到服务器
```http
POST /api/sync
Authorization: Bearer {token}
Content-Type: application/json

{
  "gameState": { ... },
  "skillsData": { ... },
  "settings": { ... }
}
```

#### 从服务器获取同步数据
```http
GET /api/sync
Authorization: Bearer {token}
```

#### 检查数据版本
```http
GET /api/sync/version?version=1
Authorization: Bearer {token}
```

### AI玩家接口 `/api/ai`

#### 获取所有AI玩家
```http
GET /api/ai
```

#### 随机获取AI玩家
```http
GET /api/ai/random?level=medium
```

#### 初始化默认AI玩家（管理员）
```http
POST /api/ai/init
Authorization: Bearer {token}
```

## 🔌 WebSocket 事件

### 连接
```javascript
const socket = io('http://localhost:3000', {
  auth: { token: 'your-jwt-token' }
});
```

### 客户端事件

| 事件 | 数据 | 说明 |
|------|------|------|
| `game:create` | - | 创建游戏房间 |
| `game:join` | `{ roomId }` | 加入游戏房间 |
| `game:move` | `{ x, y, player }` | 发送落子 |
| `game:skill` | `{ skill, target }` | 使用技能 |
| `game:end` | `{ winner, result }` | 结束游戏 |
| `game:leave` | - | 离开房间 |
| `message:send` | `{ content }` | 发送实时消息 |
| `challenge:send` | `{ targetUserId }` | 发送挑战 |
| `challenge:respond` | `{ challengerUserId, accept }` | 回应挑战 |
| `sync:data` | `{ ... }` | 同步数据 |
| `sync:request` | - | 请求同步数据 |

### 服务器事件

| 事件 | 数据 | 说明 |
|------|------|------|
| `welcome` | `{ message, user }` | 连接成功欢迎 |
| `user:online_update` | `{ userId, isOnline }` | 用户在线状态更新 |
| `user:online_list` | `{ users }` | 在线用户列表 |
| `game:created` | `{ roomId, roomData }` | 游戏房间创建 |
| `game:player_joined` | `{ player, roomData }` | 玩家加入房间 |
| `game:move_update` | `{ x, y, player, gameState }` | 落子更新 |
| `game:skill_update` | `{ userId, skill, target }` | 技能使用 |
| `game:ended` | `{ winner, result }` | 游戏结束 |
| `message:new` | `{ ... }` | 新消息 |
| `challenge:request` | `{ challenger }` | 收到挑战 |
| `challenge:response` | `{ target, accept }` | 挑战回应 |
| `sync:complete` | `{ success, timestamp }` | 同步完成 |
| `sync:data` | `{ ... }` | 同步数据 |
| `error` | `{ message }` | 错误消息 |

## 🔧 前端集成

### 1. 引入依赖

在HTML文件中引入Socket.IO和API适配层：

```html
<!-- Socket.IO -->
<script src="https://unpkg.com/socket.io-client@4.7.2/dist/socket.io.min.js"></script>

<!-- 后端API适配层 -->
<script src="gomoku-api.js"></script>
```

### 2. 初始化API

```javascript
// 创建API实例
const backendAPI = new GomokuBackendAPI({
  baseURL: 'http://your-server:3000/api',
  socketURL: 'http://your-server:3000',
  autoSync: true,
  syncInterval: 30000
});

// 设置事件回调
backendAPI.setCallbacks({
  onConnect: () => console.log('已连接'),
  onDisconnect: () => console.log('已断开'),
  onMessage: (msg) => console.log('新消息:', msg),
  onGameUpdate: (data) => console.log('游戏更新:', data),
  onUserUpdate: (data) => console.log('用户更新:', data),
  onError: (err) => console.error('错误:', err),
  onSyncComplete: (data) => console.log('同步完成:', data)
});
```

### 3. 用户登录

```javascript
// 正常登录
async function handleLogin() {
  try {
    const result = await backendAPI.login(phone, password);
    if (result.success) {
      console.log('登录成功:', result.data.user);
      // 保存用户信息到localStorage
      localStorage.setItem('gomoku_user', JSON.stringify(result.data.user));
    }
  } catch (error) {
    console.error('登录失败:', error);
  }
}

// 游客登录
async function handleGuestLogin() {
  try {
    const result = await backendAPI.guestLogin();
    if (result.success) {
      console.log('游客登录成功');
    }
  } catch (error) {
    console.error('游客登录失败:', error);
  }
}
```

### 4. 数据同步

```javascript
// 手动同步
async function syncData() {
  const gameData = {
    gameState: {
      board: game.board,
      currentPlayer: game.currentPlayer,
      moveHistory: game.moveHistory
    },
    skillsData: game.skills,
    settings: game.settings
  };
  
  await backendAPI.syncData(gameData);
}

// 自动同步已开启，无需手动调用
```

### 5. 实时游戏对战

```javascript
// 创建房间
backendAPI.createGameRoom();

// 监听房间创建
backendAPI.socket.on('game:created', (data) => {
  console.log('房间创建成功:', data.roomId);
  // 可以分享房间ID给其他玩家
});

// 加入房间
backendAPI.joinGameRoom(roomId);

// 发送落子
backendAPI.sendGameMove(x, y, 'black');

// 监听落子更新
backendAPI.socket.on('game:move_update', (data) => {
  // 更新棋盘
  game.board[data.y][data.x] = data.player === 'black' ? 1 : 2;
  renderBoard();
});
```

### 6. 管理员功能

```javascript
// 获取所有玩家
async function loadAllPlayers() {
  const result = await backendAPI.getAllPlayers(true);
  if (result.success) {
    const players = result.data.players;
    // 显示玩家列表，包含在线状态
    players.forEach(p => {
      const status = p.isOnline ? '🟢在线' : '⚪离线';
      console.log(`${p.nickname} ${status} 积分:${p.skillPoints}`);
    });
  }
}

// 发放积分
async function givePoints(phone, amount) {
  const result = await backendAPI.addPoints(phone, amount);
  if (result.success) {
    console.log('积分发放成功');
  }
}
```

## 🗄️ 数据库模型

### User（用户）
```javascript
{
  phone: String,           // 手机号
  password: String,        // 密码（加密）
  nickname: String,        // 昵称
  avatar: String,          // 头像
  role: String,            // 角色（player/admin/creator）
  isOnline: Boolean,       // 在线状态
  stats: {
    totalGames: Number,    // 总游戏数
    wins: Number,          // 胜利数
    losses: Number,        // 失败数
    skillPoints: Number    // 积分
  },
  skills: Map,             // 技能数据
  unlockedSkills: Array    // 解锁技能列表
}
```

### GameRecord（游戏记录）
```javascript
{
  gameId: String,          // 游戏ID
  players: Array,          // 玩家列表
  winner: ObjectId,        // 胜者
  result: String,          // 结果
  gameMode: String,        // 游戏模式
  moveHistory: Array,      // 落子历史
  skillsUsed: Array,       // 使用技能
  duration: Number,        // 游戏时长
  startedAt: Date,         // 开始时间
  endedAt: Date            // 结束时间
}
```

### AIPlayer（AI玩家）
```javascript
{
  aiId: String,            // AI ID
  nickname: String,        // 昵称
  avatar: String,          // 头像
  level: String,           // 等级
  stats: Object,           // 统计数据
  personality: String,     // 性格
  phrases: Object          // 发言短语
}
```

### Message（消息）
```javascript
{
  sender: ObjectId,        // 发送者
  content: String,         // 内容
  messageType: String,     // 类型
  comments: Array,         // 评论
  likes: Array,            // 点赞
  isPinned: Boolean        // 是否置顶
}
```

### SyncData（同步数据）
```javascript
{
  userId: ObjectId,        // 用户ID
  version: Number,         // 版本号
  gameState: Object,       // 游戏状态
  skillsData: Map,         // 技能数据
  settings: Object,        // 设置
  lastSyncAt: Date         // 最后同步时间
}
```

## 🔒 安全措施

1. **JWT认证**：所有需要认证的接口使用JWT令牌
2. **密码加密**：使用bcrypt加密存储密码
3. **速率限制**：防止恶意请求攻击
4. **XSS防护**：清理用户输入数据
5. **MongoDB注入防护**：防止NoSQL注入
6. **CORS配置**：限制跨域访问
7. **Helmet安全头**：设置HTTP安全头

## 📦 生产部署

### 1. 使用PM2部署

```bash
# 安装PM2
npm install -g pm2

# 启动应用
pm2 start server.js --name gomoku-backend

# 查看状态
pm2 status

# 查看日志
pm2 logs gomoku-backend
```

### 2. 使用Docker部署

创建 `Dockerfile`：

```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install --production
COPY . .
EXPOSE 3000
CMD ["node", "server.js"]
```

构建和运行：

```bash
docker build -t gomoku-backend .
docker run -p 3000:3000 gomoku-backend
```

### 3. 使用Nginx反向代理

```nginx
server {
    listen 80;
    server_name your-domain.com;

    location /api {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }

    location /socket.io {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
    }
}
```

## 🛠️ 故障排查

### MongoDB连接失败
```bash
# 检查MongoDB状态
mongod --version

# 启动MongoDB
mongod --dbpath /data/db
```

### 端口被占用
```bash
# Windows
netstat -ano | findstr :3000
taskkill /PID <pid> /F

# Linux/Mac
lsof -i :3000
kill -9 <pid>
```

### WebSocket连接失败
- 检查CORS配置
- 确认防火墙允许WebSocket
- 检查客户端token是否有效

## 📝 更新日志

### v1.0.0 (2024-01)
- 完整的用户认证系统
- RESTful API接口
- WebSocket实时通信
- 数据同步功能
- 管理员功能
- AI玩家系统

## 📄 许可证

MIT License