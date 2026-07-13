TransitSafe
Real-time school transportation management platform that enables schools to track buses, manage student ridership via QR-code scanning, and provide parents with live updates on their children's whereabouts.

Overview
TransitSafe covers the full lifecycle of school transportation: route planning, trip execution with GPS tracking, QR-code-based boarding validation, push notifications, and analytics. Schools manage their fleet and routes, drivers broadcast live GPS positions during trips, students scan QR codes to board, and parents receive real-time location updates and notifications — all through a single platform.

Key Features
Live GPS Bus Tracking — Real-time bus location on interactive maps using MapLibre GL and OpenStreetMap tiles
QR-Code Boarding Validation — Students scan HMAC-signed QR codes to verify boarding; supports offline scanning with automatic sync
Real-time Parent Notifications — Push notifications for boarding, trip start/end, delays, and absences
Multi-Role Dashboards — Separate views for School Admin, Driver, and Parent with role-based access control
Route & Trip Management — Create routes with stops, assign drivers and buses, schedule and track trips
Analytics & Reporting — Attendance tracking, trips per day, delays, and capacity utilization
Offline Support — QR scans queue locally and sync when connectivity returns; location buffering reduces database writes
Tech Stack
Layer	Technology
Framework	Next.js 16 (App Router, Server Components, Server Actions)
Language	TypeScript (strict mode)
Frontend	React 19, Tailwind CSS v4, shadcn/ui
Authentication	Clerk (JWT sessions, webhooks, user invitations)
Database	PostgreSQL (Supabase) via Prisma v7 ORM
Cache	Redis (Upstash via ioredis) — rate limiting, location cache, ETA cache
Real-time	Socket.IO — live GPS streaming, trip status, notifications
Maps	MapLibre GL JS + OpenStreetMap tiles
Push Notifications	Web Push API with VAPID keys
QR Codes	qrcode (generation) + html5-qrcode (scanning) — HMAC-SHA256 signed tokens
Validation	Zod v4
How It Works
The platform operates in a continuous loop:

Admin Setup — School admins create buses, drivers, students, and routes with stops. Drivers and parents receive Clerk invitation emails.
Trip Creation — Admin schedules a trip by selecting a route, bus, and driver. Parents and drivers are notified.
Trip Execution — Driver starts the trip from their dashboard. Their browser broadcasts GPS coordinates via Socket.IO every few seconds, which are buffered in memory and flushed to the database every 15 seconds.
Boarding — At each stop, students scan their personal QR code (printed on a card or generated in-app). The driver's scanner validates the HMAC signature, checks the trip is active, and records the boarding. Offline scans queue locally and sync when reconnected.
Live Tracking — Parents see their child's bus moving in real-time on a map, with an ETA to the next stop and to school. The admin dashboard shows all active buses fleet-wide.
Notifications — Parents receive push notifications when their child boards, when the trip starts/ends, and for any cancellations or delays.
Completion — Driver ends the trip. Attendance is recorded, analytics are updated, and outbound webhooks can be triggered.
User Roles
Role	Capabilities
Super Admin	Cross-school management, user administration, system-wide settings
School Admin	Manage buses, drivers, students, routes, trips, view analytics, send notifications
Driver	Start/end trips, broadcast GPS, scan student QR codes, manual check-in
Parent	Track child's bus in real-time, view ETA, manage absences, receive push notifications
Real-time System
The application uses Socket.IO for bidirectional communication. During an active trip, the driver's browser sends location-update events that are broadcast to two rooms: the trip room (parents tracking that specific bus) and the school room (admin fleet view). GPS pings are batched in an in-memory buffer and flushed to PostgreSQL every 15 seconds to reduce write load. The latest location per trip is cached in Redis with a 24-hour TTL for fast retrieval. For drivers with poor connectivity, QR scans are queued in sessionStorage and automatically flushed when the socket reconnects.

Security
Authentication — Clerk JWT sessions with middleware-level route protection
Authorization — Role-based access control (SUPER_ADMIN, SCHOOL_ADMIN, DRIVER, PARENT) enforced at API, server action, and Socket.IO levels
QR Tokens — HMAC-SHA256 signed payloads containing studentId and schoolId; cannot be forged without the server secret
Rate Limiting — Redis-based sliding window (60 req/min on QR validation, 10 req/min on geocoding) with in-memory fallback
Input Validation — Zod schemas on every API route and server action; string sanitization on user inputs
Audit Trail — All mutations logged with user, action, entity, and IP address
