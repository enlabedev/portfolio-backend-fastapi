# ⚡ Portfolio Backend - FastAPI (Conectador)

<div align="center">

![Python](https://img.shields.io/badge/Python-3.11-3776AB?style=for-the-badge&logo=python&logoColor=white)
![FastAPI](https://img.shields.io/badge/FastAPI-009688?style=for-the-badge&logo=fastapi&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![Cloud Run](https://img.shields.io/badge/Cloud_Run-4285F4?style=for-the-badge&logo=google-cloud&logoColor=white)

**API Backend para contacto y servicios en tiempo real**

[📖 API Docs (Swagger)](https://fastapi-backend-xxx.run.app/docs) • [🏗️ Arquitectura](#arquitectura) • [🔌 Endpoints](#endpoints)

</div>

---

## 📖 **Descripción**

Backend **FastAPI** que actúa como "conectador" del portfolio. Maneja:
- ✉️ **Formulario de contacto** (integración con Brevo)
- 📊 **Stats en tiempo real** (años experiencia, proyectos, métricas)
- 🔌 **APIs de integración** con servicios externos
- ⚡ **Endpoints ultra-rápidos** (async/await)

---

## 🏗️ **Arquitectura**

```
portfolio-backend-fastapi/
├── app/
│   ├── main.py              # Aplicación principal
│   ├── config.py            # Configuración y env vars
│   ├── models/              # Modelos Pydantic
│   │   ├── contact.py       # ContactForm model
│   │   └── stats.py         # StatsResponse model
│   ├── routers/             # Endpoints agrupados
│   │   ├── contact.py       # POST /contact
│   │   ├── stats.py         # GET /stats
│   │   └── health.py        # GET /health
│   ├── services/            # Lógica de negocio
│   │   ├── email_service.py # Brevo integration
│   │   └── stats_service.py # Stats computation
│   └── utils/               # Helpers
│       ├── logger.py
│       └── validators.py
├── tests/                   # Tests con pytest
│   ├── test_contact.py
│   ├── test_stats.py
│   └── conftest.py
├── .env.example
├── .dockerignore
├── Dockerfile
├── requirements.txt
├── pyproject.toml           # Poetry (opcional)
└── README.md
```

---

## 🚀 **Quick Start**

### **Pre-requisitos**
```bash
Python 3.12+
Poetry
Docker (para containerización)
```

### **Instalación Local**

```bash
# Clonar el repo
git clone https://github.com/enlabedev/portfolio-backend-fastapi.git
cd portfolio-backend-fastapi

# Crear virtual environment
python -m venv venv
source venv/bin/activate  # En Windows: venv\Scripts\activate

# Instalar dependencias
pip install -r requirements.txt

# Copiar variables de entorno
cp .env.example .env
nano .env  # Editar con tus credenciales
```

### **Variables de Entorno**

```env
# .env
# Brevo Email Service
BREVO_API_KEY=xkeysib-your-api-key-here
CONTACT_EMAIL=noreply@enlabedev.com
SENDER_EMAIL=noreply@enlabedev.com
SENDER_NAME=Enrique Lazo Bello

# CORS Settings
ALLOWED_ORIGINS=http://localhost:4321,https://enlabedev.com

# App Config
APP_ENV=development
DEBUG=True
LOG_LEVEL=INFO

# Rate Limiting (opcional)
RATE_LIMIT_ENABLED=True
RATE_LIMIT_PER_MINUTE=10
```

### **Ejecutar en Desarrollo**

```bash
# Modo desarrollo con hot-reload
uvicorn app.main:app --reload --port 8080

# O con el script directo
python -m app.main

# Abrir Swagger UI
# http://localhost:8080/docs
```

---

## 🔌 **Endpoints API**

### **📊 Health Check**

```http
GET /health
```

**Response:**
```json
{
  "status": "healthy",
  "version": "1.0.0",
  "uptime_seconds": 3600,
  "timestamp": "2025-12-10T14:30:00Z"
}
```

---

### **📈 Stats (Estadísticas en Tiempo Real)**

```http
GET /api/stats
```

**Response:**
```json
{
  "years_experience": 14,
  "total_projects": 45,
  "companies_worked": 12,
  "lines_of_code": "500K+",
  "languages": ["Python", "PHP", "JavaScript", "TypeScript"],
  "frameworks": ["FastAPI", "Django", "Laravel", "React"],
  "security_score": 97,
  "uptime_percentage": 99.9,
  "last_updated": "2025-12-10T14:30:00Z"
}
```

**Features:**
- ✅ Cache in-memory (Redis opcional)
- ✅ Actualización automática cada 24h
- ✅ Sin autenticación requerida

---


## 📧 **Integración con Brevo**

### **Setup de Brevo**

1. Crear cuenta en [Brevo](https://www.brevo.com/)
2. Obtener API Key desde Settings > SMTP & API
3. Configurar sender email verificado
4. Agregar API key al `.env`

### **Email Service Implementation**

```python
# app/services/email_service.py
import httpx
from app.config import settings

async def send_contact_email(name: str, email: str, message: str, contact_type: str):
    """Envía email de contacto a través de Brevo."""
    
    url = "https://api.brevo.com/v3/smtp/email"
    headers = {
        "api-key": settings.BREVO_API_KEY,
        "Content-Type": "application/json"
    }
    
    payload = {
        "sender": {
            "name": settings.SENDER_NAME,
            "email": settings.SENDER_EMAIL
        },
        "to": [
            {"email": settings.CONTACT_EMAIL, "name": "Enrique Lazo Bello"}
        ],
        "subject": f"[Portfolio] Nuevo mensaje de {name} ({contact_type})",
        "htmlContent": f"""
            <h2>Nuevo mensaje desde tu portfolio</h2>
            <p><strong>Nombre:</strong> {name}</p>
            <p><strong>Email:</strong> {email}</p>
            <p><strong>Tipo:</strong> {contact_type}</p>
            <p><strong>Mensaje:</strong></p>
            <p>{message}</p>
        """,
        "replyTo": {"email": email, "name": name}
    }
    
    async with httpx.AsyncClient() as client:
        response = await client.post(url, json=payload, headers=headers)
        response.raise_for_status()
        return response.json()
```

---

## 🧪 **Testing**

### **Ejecutar Tests**

```bash
# Todos los tests
pytest

# Con coverage
pytest --cov=app --cov-report=html

# Tests específicos
pytest tests/test_contact.py -v

# Tests con logs
pytest -s
```

---

## 📊 **Monitoreo**

### **Logs en Cloud Run**

```bash
# Ver logs en tiempo real
gcloud logging read "resource.type=cloud_run_revision AND resource.labels.service_name=fastapi-backend" --limit 50 --format json
```

### **Métricas Clave**

- Request count
- Response time (p50, p95, p99)
- Error rate
- CPU/Memory usage
- Cold starts

---

## 📚 **Documentación Adicional**

- [📖 FastAPI Docs](https://fastapi.tiangolo.com)
- [📧 Brevo API Docs](https://developers.brevo.com)
- [☁️ Cloud Run Docs](https://cloud.google.com/run/docs)

---

## 🤝 **Contribuciones**

Ver guía en el [repo principal](https://github.com/enlabedev/portfolio-infra).

---

## 👨‍💻 **Autor**

**Enrique Lazo Bello** - Senior Software Engineer

[![LinkedIn](https://img.shields.io/badge/LinkedIn-0077B5?style=for-the-badge&logo=linkedin&logoColor=white)](https://linkedin.com/in/enlabe)
[![GitHub](https://img.shields.io/badge/GitHub-100000?style=for-the-badge&logo=github&logoColor=white)](https://github.com/enlabedev)

---

## 📄 **Licencia**

MIT License

---

<div align="center">

Made with ⚡ FastAPI and ❤️

</div>
