# Lom Wong Café & Restaurant

Full-stack web application for **Lom Wong Café & Restaurant (ล้อมวง คาเฟ่)**.

## Demo

- Live demo: `http://lomwong.me`
- Dashboard login: `http://lomwong.me/dashboard/login`
- Pivot worksheet: [PIVOT.md](./PIVOT.md)

Seeded demo accounts:

- Owner: `owner@lomwong.local` / `owner1234`
- Staff: `staff@lomwong.local` / `staff1234`

## Project Tree

```text
lomwong-cafe/
├── frontend/
│   ├── app/
│   │   ├── page.js
│   │   ├── menu/page.js
│   │   ├── booking/page.js
│   │   ├── contact/page.js
│   │   ├── order/[token]/page.js
│   │   └── dashboard/
│   │       ├── page.js
│   │       ├── login/page.js
│   │       └── orders/new/page.js
│   ├── components/
│   │   ├── Navbar.jsx
│   │   ├── StatusBadge.jsx
│   │   └── ChatbotWidget.jsx
│   ├── lib/api.js
│   ├── package.json
│   ├── tailwind.config.js
│   └── next.config.js
├── backend/
│   ├── data/lomwong_cafe_kb.txt
│   ├── middleware/auth.js
│   ├── routes/
│   │   ├── ai.js
│   │   ├── auth.js
│   │   ├── bookings.js
│   │   ├── menu.js
│   │   ├── orders.js
│   │   ├── promotions.js
│   │   ├── qr.js
│   │   ├── sales.js
│   │   ├── staff.js
│   │   └── store.js
│   ├── services/ai.js
│   ├── utils/prisma.js
│   ├── package.json
│   └── server.js
├── prisma/
│   ├── schema.prisma
│   └── seed.js
├── nginx/lomwong.conf
├── docker-compose.yml
├── ecosystem.config.js
├── .env.example
└── README.md
```

## Features

- Customer site: home, live store status, menu, today-only booking, contact/map.
- Floating โมจิ chatbot using the Gemini API with `backend/data/lomwong_cafe_kb.txt`.
- Owner/staff dashboard with JWT login.
- Store status manager, menu CRUD, sold-out menu status, promotion CRUD, booking manager, and staff manager.
- Auto sales logging, daily/weekly/monthly summary, and โมจิ Alert when daily sales are below threshold.
- Caption Generator with goal, audience, platform, tone, length, hashtag count, and copy-ready outputs.
- QR ordering and staff order entry with status flow `pending -> preparing -> ready -> paid`.
- Room information manager for daily rooms, room availability, and room price.

## Demo Day Polish Checklist

### UI / UX

- [x] ชื่อ app และ branding สอดคล้องกับ domain ใหม่: Lom Wong Café & Daily Rooms
- [x] ภาษาที่ใช้เหมาะกับลูกค้าร้านอาหาร/ห้องพักและพนักงานร้าน
- [x] มี error message ภาษาไทยที่อ่านเข้าใจง่ายใน booking, login, ordering, และ dashboard
- [x] หน้าตาเป็น Next.js dashboard สะอาด ไม่มี debug output โผล่ในหน้าเว็บ

### Code Quality

- [x] ไม่มี debug `print()` ในโค้ด เพราะโปรเจกต์ใช้ Node.js/Next.js
- [x] ไม่มี hard-coded API key ในโค้ด และใช้ `.env` / `.env.example`
- [x] dependencies ครบใน `package.json`, `frontend/package.json`, และ `backend/package.json`

### README.md

- [x] README อธิบายระบบสำหรับ domain ล้อมวง ไม่ใช่ MilkLab°
- [x] มี link ไปยัง live demo URL
- [x] มีวิธีรันในเครื่องท้องถิ่นด้วย Docker Desktop และ manual local setup
- [x] มี link ไปยัง [PIVOT.md](./PIVOT.md)

## Local Development

### Docker Desktop

Run the full stack with PostgreSQL, backend, and frontend:

```bash
docker compose up -d --build
```

Open:

- Frontend: `http://localhost:3000`
- Backend health: `http://localhost:4000/api/health`

Useful Docker commands:

```bash
docker compose ps
docker compose logs -f backend
docker compose down
```

The compose setup uses `.env.example` as defaults and overrides `DATABASE_URL` for the internal Docker network. Create a real `.env` later when adding production secrets such as `JWT_SECRET`, `GEMINI_API_KEY`, Google Sheets, or Telegram credentials.

### Manual Local Setup

1. Copy env file:

```bash
cp .env.example .env
```

2. Edit `.env`:

```env
DATABASE_URL=postgresql://lomwong:lomwong@localhost:5432/lomwong
JWT_SECRET=replace_with_a_long_secret
GEMINI_API_KEY=your_key_here
GEMINI_MODEL=gemini-2.5-flash
GOOGLE_SHEET_ID=your_google_sheet_id
GOOGLE_SHEET_WORKSHEET=Sales
GOOGLE_APPLICATION_CREDENTIALS=./google-service-account.json
TELEGRAM_BOT_TOKEN=your_telegram_bot_token
TELEGRAM_CHAT_ID=your_telegram_chat_id
STORE_ALERT_THRESHOLD=500
NEXT_PUBLIC_API_URL=http://localhost:4000
FRONTEND_ORIGIN=http://localhost:3000
```

3. Start PostgreSQL:

```bash
docker compose up -d
```

4. Install dependencies:

```bash
npm run install:all
```

5. Generate Prisma client and migrate:

```bash
npm run db:generate
npm run db:migrate -- --name init
npm run db:seed
```

6. Run the app:

```bash
npm run dev
```

Open:

- Frontend: `http://localhost:3000`
- Backend health: `http://localhost:4000/api/health`

Seeded logins are listed in the Demo section above.

## API Summary

Auth:

- `POST /api/auth/login`
- `POST /api/auth/logout`

Store:

- `GET /api/store/status`
- `PATCH /api/store/status`

Menu:

- `GET /api/menu`
- `POST /api/menu`
- `PUT /api/menu/:id`
- `DELETE /api/menu/:id`

Promotions:

- `GET /api/promotions`
- `POST /api/promotions`
- `PUT /api/promotions/:id`
- `DELETE /api/promotions/:id`

Bookings:

- `POST /api/bookings`
- `GET /api/bookings/today`
- `PATCH /api/bookings/:id/status`

Orders:

- `POST /api/orders`
- `GET /api/orders`
- `PATCH /api/orders/:id/status`

QR:

- `POST /api/qr/generate`
- `GET /api/qr/generate`
- `GET /api/qr/validate/:token`

Sales:

- `POST /api/sales`
- `GET /api/sales/summary?period=day|week|month`
- `POST /api/sales/demi-alert`

Staff:

- `GET /api/staff`
- `POST /api/staff`
- `PUT /api/staff/:id`
- `DELETE /api/staff/:id`

AI:

- `POST /api/chat`
- `POST /api/ai/caption`

## DigitalOcean Droplet Setup

Ubuntu 22.04 commands:

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y nginx postgresql postgresql-contrib git build-essential
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs
sudo npm install -g pm2
```

Create database:

```bash
sudo -u postgres psql
CREATE USER lomwong WITH PASSWORD 'strong_password_here';
CREATE DATABASE lomwong OWNER lomwong;
\q
```

Deploy code:

```bash
cd /var/www
sudo git clone https://github.com/yourname/lomwong-cafe.git
sudo chown -R $USER:$USER /var/www/lomwong-cafe
cd /var/www/lomwong-cafe
cp .env.example .env
nano .env
```

Install, migrate, and build:

```bash
npm run install:all
npm run db:generate
npm run db:migrate -- --name init
npm run db:seed
npm --prefix frontend run build
```

Start with PM2:

```bash
pm2 start ecosystem.config.js
pm2 save
pm2 startup
```

Configure Nginx:

```bash
sudo cp nginx/lomwong.conf /etc/nginx/sites-available/lomwong.conf
sudo ln -s /etc/nginx/sites-available/lomwong.conf /etc/nginx/sites-enabled/lomwong.conf
sudo nginx -t
sudo systemctl reload nginx
```

Enable SSL with Certbot:

```bash
sudo apt install -y certbot python3-certbot-nginx
sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com
```

Useful commands:

```bash
pm2 status
pm2 logs lomwong-backend
pm2 logs lomwong-frontend
sudo systemctl reload nginx
```

## Production Notes

- Replace `JWT_SECRET` with a long random value.
- Set `GEMINI_API_KEY` before using chatbot/caption generation in production.
- For Session 2 style Sales Logger, set `GOOGLE_SHEET_ID`, `GOOGLE_SHEET_WORKSHEET`, and `GOOGLE_APPLICATION_CREDENTIALS`; new sales will still save to PostgreSQL and will also append to Google Sheets when configured.
- For Telegram โมจิ Alert, set `TELEGRAM_BOT_TOKEN` and `TELEGRAM_CHAT_ID`, then call `POST /api/sales/demi-alert` from the dashboard, a cron job, or GitHub Actions.
- Update `NEXT_PUBLIC_API_URL` and `FRONTEND_ORIGIN` to your domain.
- The backend accepts menu images via multipart `image` upload or an `imageUrl` field.
- Dashboard auth is stored in browser localStorage; for stricter production security, move JWT to httpOnly cookies.

## Demo Day Self-Check

- [x] Deploy URL ใช้งานได้ (เปิดทดสอบล่าสุด: 2026-05-25)
- [x] ไม่มี `.env` ใน git history และไม่มี secret JSON ถูก commit
- [x] PIVOT.md ครบ 3 ข้อ
- [x] README อธิบายระบบของ domain ตัวเอง (ไม่ใช่ MilkLab°)
- [x] knowledge base, prompt, UI ปรับเป็น domain ใหม่หมดแล้ว
