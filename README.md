# 🩺 DermaMED

> **Practice Management System (PMS) para un consultorio dermatológico privado en Argentina.**

DermaMED es un sistema de gestión integral diseñado para digitalizar y centralizar todas las operaciones de un consultorio dermatológico: turnos, historia clínica, facturación, stock de productos y comunicación con pacientes vía WhatsApp.

---

## 📌 Contexto y Problema que Resuelve

El consultorio operaba con una combinación de papel, WhatsApp y planillas de Excel para gestionar turnos, registros médicos e inventario. Esto generaba:

- Pérdida de información y doble carga de datos
- Alta tasa de ausentismo por falta de recordatorios automáticos
- Dificultad para tener visibilidad del stock en tiempo real
- Sin historial clínico digital estructurado

**DermaMED reemplaza todo eso con un sistema centralizado**, convirtiéndose en la **Única Fuente de Verdad** del calendario y el stock del consultorio.

---

## 👥 Usuarios del Sistema

| Usuario | Rol | Necesidades principales |
|---|---|---|
| **Dra. Roxana Amante** | Médica dermatóloga / Dueña | Métricas, control de stock, historia clínica |
| **Mariela** | Recepcionista | Agilidad operativa, recordatorios automáticos, trato humano |

### Filosofía "Mariela First"

> Cada feature se evalúa con la pregunta: **¿Le ahorra un click a Mariela o le agrega uno?**

El sistema **no** busca reemplazar a la recepcionista con autogestión del paciente, sino potenciarla. El flujo de turnos es **híbrido-asistido**:

1. El paciente contacta por WhatsApp
2. Mariela acuerda el horario manualmente
3. Mariela carga el turno en el sistema
4. **El sistema dispara automáticamente** la confirmación y el recordatorio 24 h antes vía WhatsApp API

---

## 🏗️ Arquitectura General

El proyecto está dividido en **3 repositorios Git independientes**, orquestados por Docker Compose:

```
/Proyectos
├── backend/     → API REST (Laravel 12)
├── frontend/    → SPA / Dashboard (Next.js 16)
└── setup/       → Infraestructura (Docker Compose)
```

> ⚠️ El directorio raíz **no** es un repositorio Git. Cada subdirectorio tiene su propio `git`.

### Servicios en desarrollo local

| Servicio | URL |
|---|---|
| Backend API | http://localhost:8080/api |
| Frontend | http://localhost:3000 |
| Adminer (DB GUI) | http://localhost:8081 |
| PostgreSQL | localhost:5432 |

---

## 🛠️ Stack Tecnológico

### Backend

| Tecnología | Versión | Rol |
|---|---|---|
| PHP | 8.2+ | Lenguaje base |
| Laravel | 12 | Framework principal |
| PostgreSQL | 17 | Base de datos relacional |
| JWT (`tymon/jwt-auth`) | 2.2 | Autenticación stateless |
| Resend (`resend-php`) | — | Envío de emails transaccionales |
| Laravel Pint | — | Formateo y calidad de código |
| PHPUnit | 11 | Testing |

### Frontend

| Tecnología | Versión | Rol |
|---|---|---|
| Next.js (App Router) | 16 | Framework React con SSR |
| React | 19 | Librería de UI |
| TypeScript | 5 (strict) | Tipado estático |
| Tailwind CSS | v4 | Estilos utilitarios |
| TanStack Query | 5 | Server state management |
| React Hook Form + Zod | — | Formularios y validación |
| Radix UI | — | Componentes accesibles (base) |
| Lucide React | — | Iconografía |
| Axios | — | Cliente HTTP |
| Sonner | — | Notificaciones toast |
| date-fns | 4 | Manejo de fechas |

### Infraestructura

| Tecnología | Rol |
|---|---|
| Docker + Docker Compose | Contenedores para todos los servicios |
| Adminer | GUI web para administrar la base de datos |

---

## 🗃️ Modelo de Datos (Resumen)

El modelo está centrado en el **paciente** y sus interacciones con el consultorio:

- **`patients`** — Datos demográficos y contacto del paciente
- **`appointments`** — Turnos, vinculados a paciente, médico y servicio
- **`medical_records`** — Historia clínica (inmutable, sin hard delete)
- **`invoices` / `invoice_items` / `invoice_payments`** — Facturación completa
- **`products` / `stock_movements`** — Control de inventario
- **`services`** — Catálogo de servicios del consultorio
- **`users` / `roles`** — Gestión de usuarios con roles muchos a muchos
- **`doctor_availability`** — Horarios disponibles del médico
- **`health_insurances`** — Obras sociales

---

## 🔐 Seguridad y Cumplimiento (Standard Healthcare)

Al manejar datos de salud sensibles, el sistema sigue reglas estrictas:

1. **Cero fugas de datos**: El backend nunca envía datos si el usuario no tiene permiso. No se usa CSS para ocultar info sensible.
2. **Roles múltiples**: Relación muchos a muchos entre usuarios y roles. La validación siempre verifica membresía en array, nunca igualdad estricta.
3. **Inmutabilidad clínica**: Prohibido el `DELETE` físico en tablas críticas (`patients`, `appointments`, `medical_records`, `invoices`). Se usa siempre `SoftDeletes`.
4. **Privilegio mínimo**: Toda ruta de API y componente de frontend está bloqueado por defecto. Solo se expone si se verifica el rol explícitamente.

---

## 📋 Módulos del Sistema

| Módulo | Descripción |
|---|---|
| **Dashboard** | Vista general con métricas clave del consultorio |
| **Pacientes** | Ficha de paciente, historial de turnos e historia clínica |
| **Turnos** | Agenda del consultorio con gestión de disponibilidad |
| **Facturación** | Creación y seguimiento de facturas con múltiples métodos de pago |
| **Productos** | Inventario de productos con movimientos de stock |
| **Servicios** | Catálogo de prestaciones médicas y estéticas |
| **Usuarios** | Alta y gestión de personal del consultorio con roles |

---

## 🚀 Levantar el Entorno de Desarrollo

El repositorio principal usa **git submodules** para incluir los tres repos (`backend`, `frontend`, `setup`). Es importante clonar con el flag correcto para que todo quede en un solo paso.

```bash
# 1. Clonar el repo principal junto con todos los submódulos
git clone --recurse-submodules https://github.com/nacho1706/dermamed.git

# Si ya clonaste sin el flag, inicializá los submódulos manualmente
git submodule update --init --recursive
```

```bash
# 2. Configurar variables de entorno del backend
cp backend/.env.example backend/.env
# Editá el .env con tus claves locales (DB, JWT, etc.)

# 3. Levantar todos los servicios
cd setup
docker compose up -d

# 4. Correr migraciones (y seeders si es primera vez)
docker exec backend-dermamed php artisan migrate --seed
```

> **Requisito**: Tener Docker y Docker Compose instalados. No se requiere PHP, Node o Composer en el host.

---

## 📝 Convenciones de Código

- **Idioma del código**: Todo en **inglés** (rutas, nombres de variables, archivos). Solo el texto visible para el usuario va en español.
- **Git Workflow**: `main` → `develop` → `feature/<nombre>` / `fix/<nombre>`
- **Commits**: [Conventional Commits](https://www.conventionalcommits.org/) — ej: `feat(patients): add search`, `fix(auth): handle expired token`

---

## 📁 Glosario

| Término | Significado |
|---|---|
| PMS | Practice Management System |
| EHR | Electronic Health Record (Historia Clínica Electrónica) |
| CUIT | Identificador fiscal único en Argentina |
| Recordatorio 24h | Mensaje automático enviado el día previo al turno |
| Soft Delete | Borrado lógico; el registro se marca como eliminado pero permanece en la BD |
