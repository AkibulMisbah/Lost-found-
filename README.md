# Lost & Found Campus Microservices System

A complete microservices-based Lost & Found system for college campuses featuring real-time matching, notifications, and a professional React frontend.

## 🏗️ Architecture

```
Frontend (React)
     ↓
API Gateway (Port 3000)
     ↓
┌────────────────────────────────────────────────┐
│  Users Service (4001)                           │
│  Lost Items Service (4002)                      │
│  Found Items Service (4003)                     │
│  Matching Service (4004)                        │
│  Notifications Service (4005)                   │
└────────────────────────────────────────────────┘
     ↓
MongoDB (separate DB per service)
```

## 🚀 Quick Start

### Prerequisites
- Node.js 18+
- Docker & Docker Compose
- npm (v8+)

### Setup

1. **Install dependencies:**
   ```bash
   npm install
   ```

2. **Start services with Docker Compose:**
   ```bash
   docker-compose up -d
   ```

3. **Start frontend dev server:**
   ```bash
   cd packages/frontend
   npm run dev
   ```

4. **Access the app:**
   - Frontend: http://localhost:5173
   - API Gateway: http://localhost:3000

### Verify Services
```bash
curl http://localhost:3000/health
curl http://localhost:4001/health
curl http://localhost:4002/health
```

## 📦 Services

### API Gateway (Port 3000)
- Express server routing all requests
- JWT authentication
- Correlation ID logging
- WebSocket proxy to notifications service

### Users Service (Port 4001)
- User registration & login
- JWT token generation
- Password hashing (bcrypt)
- Profile management

### Lost Items Service (Port 4002)
- Create/read/update/delete lost items
- Publishes events to matching service
- Categories: electronics, documents, accessories, clothing, keys, bags

### Found Items Service (Port 4003)
- Create/read/update/delete found items
- Publishes events to matching service

### Matching Service (Port 4004)
- Listens to item creation events
- Matching algorithms:
  - **Exact match**: category + description match
  - **Fuzzy match**: text similarity (>70% threshold)
  - **Location-based**: same campus zone
- Publishes `match.found` events when matches detected

### Notifications Service (Port 4005)
- Stores notifications in MongoDB
- WebSocket server (Socket.io) for real-time updates
- Receives match events and broadcasts to users
- Mark notifications as read

## 🎨 Frontend Features

- **Login/Register** - Secure JWT-based authentication
- **Dashboard** - View all lost/found items with filters
- **Post Items** - Create lost or found item reports
- **Real-time Notifications** - WebSocket-powered instant match alerts
- **Professional UI** - Clean, minimal design (no gradients)
- **Responsive Design** - Mobile-friendly layout

## 📋 Database Schema

### User
- `_id`: ObjectId
- `name`: string
- `email`: string (unique)
- `passwordHash`: string
- `location`: string
- `createdAt`, `updatedAt`: Date

### Lost Item
- `_id`: ObjectId
- `title`, `description`: string
- `category`: string
- `location`: string
- `photo_url`: string
- `userId`: string
- `status`: 'lost' | 'found_match' | 'recovered'
- `createdAt`, `updatedAt`: Date

### Found Item
- Similar structure to Lost Item
- `status`: 'unclaimed' | 'claimed' | 'returned'

### Notification
- `_id`: ObjectId
- `userId`: string
- `type`: 'match_found' | 'item_claimed' | 'item_recovered'
- `data`: object (flexible)
- `read`: boolean
- `createdAt`: Date

## 🔄 Event Flow

```
1. User posts lost item
   ↓
2. Lost Items Service makes HTTP POST to Matching Service
   ↓
3. Matching Service searches for matches
   ↓
4. Match found → makes HTTP POST to Notifications Service
   ↓
5. Notifications Service stores notification in MongoDB
   ↓
6. WebSocket broadcast to matching user
   ↓
7. Real-time notification appears in UI
```

## 🛠️ Development

### Run all services in dev mode:
```bash
# From root directory, install all workspaces first
npm install

# Then run individual services in separate terminals
npm run dev:gateway
npm run dev:users
npm run dev:lost-items
npm run dev:found-items
npm run dev:matching
npm run dev:notifications

# In another terminal
npm run dev:frontend
```

### Test a service individually:
```bash
cd packages/users-service
npm run dev
```

### View logs with correlation IDs:
```bash
docker-compose logs api-gateway
docker-compose logs matching-service
```

### Stop services:
```bash
docker-compose down
```

## 📚 API Endpoints

### Auth
- `POST /auth/register` - Create account
- `POST /auth/login` - Get JWT token
- `GET /auth/profile` - Get current user

### Items
- `GET /lost` - List all lost items
- `POST /lost` - Create lost item (auth required)
- `GET /lost/:id` - Get lost item details
- `GET /found` - List all found items
- `POST /found` - Create found item (auth required)
- `GET /found/:id` - Get found item details

### Matches
- `GET /matches/:itemId` - Get matches for an item

### Notifications
- `GET /notifications` - Get user notifications (auth required)
- `PUT /notifications/:id/read` - Mark notification as read

### WebSocket
- Connect: `http://localhost:3001` with JWT auth
- Events: `notification` (real-time match updates)

## 🔐 Environment Variables

See `.env.example` for all available options.

## 📞 Support

For issues or questions, check logs:
```bash
docker-compose logs --tail=50 [service-name]
```

## 📝 License

MIT
