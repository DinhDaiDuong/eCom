# Sử dụng Node.js để build React & Express
FROM node:16 AS build
WORKDIR /app

# Cài đặt dependencies frontend
COPY frontend/package.json frontend/package-lock.json ./frontend/
RUN cd frontend && npm install

# Build frontend
COPY frontend ./frontend
RUN cd frontend && npm run build

# Cài đặt dependencies backend
COPY backend/package.json backend/package-lock.json ./backend/
RUN cd backend && npm install

# Copy toàn bộ code backend
COPY backend ./backend

# Chạy backend và serve frontend
WORKDIR /app/backend
ENV PORT=5000
EXPOSE 5000
CMD ["node", "server.js"]
