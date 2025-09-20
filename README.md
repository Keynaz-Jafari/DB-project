
# Setup Guide

## Project Structure

```

DB-project
├── Project-Backend/      # Old Django backend (with Docker + MySQL setup)
├── backend/              # New Django backend (with Docker + MySQL setup)
├── src/                  # React frontend source code
├── public/               # Frontend assets
├── package.json          # Frontend dependencies
├── vite.config.js        # Frontend Vite config
└── README.md             # This file

```

---

## 1. Run the Backend and Database

Both `Project-Backend/` and `backend/` contain a Django + MySQL stack with Docker.  
You can choose **either backend folder** depending on your setup (they are structurally the same).  

### Prerequisites
- Install **Docker Engine** and **Docker Compose**.

### Steps
1. Clone the repository and switch to the correct branch.
2. Navigate into the backend folder you want to use:
```bash
   cd backend
```

or

```bash
cd Project-Backend
```

3. Build the backend Docker image:

   ```bash
   docker build -t my-django-app .
   ```

   Replace `my-django-app` with a name of your choice.

4. Open `docker-compose.yml` in the same folder and set your image name under the `app` service:

   ```yaml
   app:
     image: my-django-app   # <---- use your image name here
     restart: unless-stopped
     depends_on:
       db:
         condition: service_healthy
   ```

5. Create a `.env` file in the same backend folder:

   ```env
   # --- MySQL container creds ---
   MYSQL_ROOT_PASSWORD=rootpass
   MYSQL_DATABASE=university
   MYSQL_USER=uniapp
   MYSQL_PASSWORD=uniapp

   # --- Django settings ---
   DJANGO_SECRET=change-me
   DB_NAME=${MYSQL_DATABASE}
   DB_USER=${MYSQL_USER}
   DB_PASSWORD=${MYSQL_PASSWORD}
   DB_HOST=db
   DB_PORT=3306
   ```

6. Start the services:

   ```bash
   docker compose --env-file .env up -d
   ```

✅ Your Django backend and MySQL database should now be running.

---

## 2. Run the Frontend

The React frontend is in the root of the repository.

### Prerequisites

* Install **Node.js 18+** from [nodejs.org](https://nodejs.org/).

### Steps

1. From the repo root:

   ```bash
   npm install
   ```

2. API requests are proxied to the Django backend through `vite.config.js`:

   ```js
   proxy: {
     "/api": {
       target: "http://localhost:8000",
       changeOrigin: true,
       secure: false,
     },
   }
   ```

   * If your backend runs on a different port/host, update the `target`.

3. Add your frontend port (`5173` by default) to Django’s `CSRF_TRUSTED_ORIGINS` in `settings.py`:

   ```python
   CSRF_TRUSTED_ORIGINS = [
       "http://localhost:5173",
       "http://localhost:5186",
       "http://localhost:5187",
       "http://localhost:5188",
       "http://localhost:5189",
   ]
   ```

4. Run the development server:

   ```bash
   npm run dev
   ```

   * The app will be available at [http://localhost:5173](http://localhost:5173).

---

## 3. Summary

* **Backend + Database**: choose `backend/` or `Project-Backend/`, configure `.env`, and run with Docker.
* **Frontend**: install dependencies, configure backend proxy, and run with Vite.
* **Connection**: ensure frontend port is listed in Django’s `CSRF_TRUSTED_ORIGINS`.



