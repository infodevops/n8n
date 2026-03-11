# n8n Setup with Docker in WSL (Ubuntu 24.04)

Production-style setup for running **n8n** inside **WSL (Ubuntu 24.04)** using **Docker** and **PostgreSQL**, accessible from a Windows browser.

## Architecture

```
Windows 11
   │
Browser → http://localhost:5678
   │
WSL2 (Ubuntu 24.04)
   │
Docker
   │
Docker Compose
   │
n8n + PostgreSQL
```

---

# 1. Install WSL

Open **PowerShell as Administrator**

```powershell
wsl --install
```

Restart the computer if prompted.

Install Ubuntu 24.04 from Microsoft Store or via WSL.

Verify installation:

```powershell
wsl -l -v
```

---

# 2. Enable systemd in WSL

Shutdown WSL:

```powershell
wsl --shutdown
```

Edit the configuration inside Ubuntu:

```bash
sudo nano /etc/wsl.conf
```

Add:

```
[boot]
systemd=true
```

Restart WSL again:

```powershell
wsl --shutdown
```

---

# 3. Install Docker and Docker Compose

Inside Ubuntu:

```bash
sudo apt update
sudo apt install -y docker.io docker-compose-v2
```

Enable Docker service:

```bash
sudo systemctl enable docker
sudo systemctl start docker
```

Add your user to the Docker group:

```bash
sudo usermod -aG docker $USER
```

Restart WSL:

```powershell
wsl --shutdown
```

Open Ubuntu again.

Verify Docker:

```bash
docker run hello-world
```

---

# 4. Create Project Folder

```
mkdir ~/n8n-docker
cd ~/n8n-docker
```

---

# 5. Create Docker Compose File

Create:

```
nano docker-compose.yml
```

Paste:

```yaml
version: "3.8"

services:

  postgres:
    image: postgres:15
    restart: always
    environment:
      POSTGRES_USER: root
      POSTGRES_PASSWORD: Testing.123
      POSTGRES_DB: n8n
    volumes:
      - postgres_data:/var/lib/postgresql/data

  n8n:
    image: docker.n8n.io/n8nio/n8n
    restart: always
    ports:
      - "5678:5678"
    dns:
      - "8.8.8.8"
      - "8.8.4.4"
    environment:
      DB_TYPE: postgresdb
      DB_POSTGRESDB_HOST: postgres
      DB_POSTGRESDB_DATABASE: n8n
      DB_POSTGRESDB_USER: root
      DB_POSTGRESDB_PASSWORD: Testing.123
      N8N_PORT: 5678
      N8N_PROTOCOL: https
      WEBHOOK_URL: https://bonzer-nanci-gablelike.ngrok-free.dev
      N8N_EMAIL_MODE: smtp
      N8N_SMTP_HOST: smtp.gmail.com
      N8N_SMTP_PORT: 587
      N8N_SMTP_USER: 
      N8N_SMTP_PASS: 
      N8N_SMTP_SENDER: 
      N8N_SMTP_SSL: "false"
      N8N_SMTP_TLS: "true"
    volumes:
      - n8n_data:/home/node/.n8n
    depends_on:
      - postgres

volumes:
  n8n_data:
  postgres_data:
```

---

# 6. Start the Containers

Run:

```bash
docker compose up -d
```

Check containers:

```bash
docker ps
```

Expected containers:

* n8n
* postgres

---

# 7. Access n8n from Windows

Open browser:

```
http://localhost:5678
```

You should see the **n8n workflow editor UI**.

---

# 8. Useful Commands

### Stop containers

```bash
docker compose down
```

### Start containers

```bash
docker compose up -d
```

### View logs

```bash
docker compose logs -f
```

### Restart containers

```bash
docker compose restart
```

---

# 9. Data Persistence

Docker volumes store all data:

```
postgres_data
n8n_data
```

Your workflows and database survive container restarts.

---

# 10. Access From Local Network (Optional)

Find your Windows IP:

```powershell
ipconfig
```

Example:

```
192.168.1.50
```

Access from another device:

```
http://192.168.1.50:5678
```

---

# Folder Structure

```
n8n-docker
 ├─ docker-compose.yml
 └─ README.md
```

---

# Recommended Future Improvements

* Reverse proxy (Nginx or Traefik)
* HTTPS with Let's Encrypt
* Redis queue mode for high workloads
* Custom domain
* Backup automation

---

# References

* https://docs.n8n.io
* https://docs.docker.com
* https://learn.microsoft.com/windows/wsl
