<p align="center">
  <img src="https://capsule-render.vercel.app/api?type=waven&color=1793D1&height=180&section=header&text=Monitor-Perifericos&fontColor=FFFFFF&fontSize=42&fontFamily=JetBrains+Mono" alt="Monitor-Perifericos" />
</p>

<h3 align="center">Sistema de monitoreo de periféricos y puertos USB en Windows</h3>

<p align="center">
  Proyecto Final de <b>Desarrollo de Software IV</b> · Universidad Tecnológica de Panamá
</p>

<p align="center">
  <img src="https://img.shields.io/badge/.NET-8.0-512BD4?style=flat-square&logo=dotnet&logoColor=white" alt=".NET 8" />
  <img src="https://img.shields.io/badge/C%23-C%23_12-239120?style=flat-square&logo=csharp&logoColor=white" alt="C#" />
  <img src="https://img.shields.io/badge/ASP.NET_Core-Web_API-512BD4?style=flat-square&logo=dotnet&logoColor=white" alt="ASP.NET Core" />
  <img src="https://img.shields.io/badge/SQL_Server-MonitorPuertos-CC2927?style=flat-square&logo=microsoftsqlserver&logoColor=white" alt="SQL Server" />
  <img src="https://img.shields.io/badge/WMI-Win32-0078D4?style=flat-square&logo=windows&logoColor=white" alt="WMI" />
</p>

---

## 📋 Descripción

**Monitor-Perifericos** es un sistema que vigila en tiempo real los **periféricos y puertos USB** de una máquina Windows. Detecta conexiones y desconexiones (USB, monitores, dispositivos de sonido, discos extraíbles, ratones), registra cada evento en **SQL Server** y lo expone mediante una **API REST** con un **dashboard web** para consultar la actividad histórica.

## 🏗️ Arquitectura

El proyecto está compuesto por tres piezas que trabajan juntas:

| Componente | Tecnología | Función |
|---|---|---|
| **MonitorService** | .NET 8 · Windows Service (BackgroundService) | Escanea los periféricos vía **WMI** cada 5 segundos, detecta cambios y los persiste en la base de datos. |
| **MonitorAPI** | .NET 8 · ASP.NET Core Web API | Expone la información por REST, sirve el dashboard web y genera reportes/exportaciones. |
| **MonitorPuertos** | SQL Server · T-SQL | Base de datos con las tablas de periféricos, puertos y su historial de actividad. |

```
┌──────────────────┐     WMI (Win32_*)     ┌──────────────┐
│  MonitorService  │ ───────────────────▶  │    Windows   │
│  (Windows Svc)   │   USB, monitores,     │   (hardware) │
└────────┬─────────┘   sonido, discos...   └──────────────┘
         │  registra eventos cada 5 s
         ▼
┌──────────────────┐     queries Dapper    ┌──────────────┐
│    SQL Server    │ ◀──────────────────── │  MonitorAPI  │
│  (MonitorPuertos)│                       │  (REST+Web)  │
└──────────────────┘                       └──────┬───────┘
                                                  │ HTTP
                                                  ▼
                                          ┌──────────────┐
                                          │  Dashboard   │
                                          │     web      │
                                          └──────────────┘
```

## 📦 Estructura

```
Monitor-Perifericos/
├── MonitorService/          # Servicio de Windows (.NET 8)
│   ├── Services/
│   │   ├── PeripheralMonitor.cs   # Escaneo vía WMI
│   │   └── DatabaseService.cs     # Persistencia en SQL Server
│   ├── MonitorWorker.cs           # Ciclo de monitoreo (BackgroundService)
│   └── Program.cs
├── MonitorAPI/              # API REST + dashboard (.NET 8)
│   ├── Controllers/         # Peripherals, Ports, Reports
│   ├── Services/
│   ├── wwwroot/             # Frontend web (HTML/CSS/JS)
│   └── Program.cs
├── MonitorPuertos.sql       # Esquema de la base de datos
├── Instalar.ps1             # Instala el servicio de Windows
├── Configurar.ps1           # Configura la base de datos
├── Iniciar.ps1              # Inicia el servicio
└── Proyecto Final.sln
```

## 🗄️ Base de datos

La BD `MonitorPuertos` (SQL Server, en `localhost` con seguridad integrada) tiene 5 tablas:

- **TiposPerifericos** — catálogo de tipos de dispositivo
- **Perifericos** — periféricos detectados
- **ActividadPerifericos** — historial de conexiones/desconexiones
- **Puertos** — puertos físicos monitoreados
- **UsoPuertos** — historial de uso de cada puerto

## 🔌 Endpoints de la API

La API corre en `https://localhost:62063` / `http://localhost:62064` y documenta sus rutas con **Swagger**.

### Periféricos
| Método | Ruta | Descripción |
|---|---|---|
| GET | `/api/Peripherals` | Lista todos los periféricos |
| GET | `/api/Peripherals/active` | Periféricos activos |
| GET | `/api/Peripherals/{id}` | Detalle de un periférico |
| GET | `/api/Peripherals/{id}/activity` | Actividad de un periférico |
| GET | `/api/Peripherals/dashboard/stats` | Estadísticas del dashboard |

### Puertos
| Método | Ruta | Descripción |
|---|---|---|
| GET | `/api/Ports/physical` | Puertos físicos |
| GET | `/api/Ports/recent` | Conexiones recientes |
| POST | `/api/Ports/{id}/verify` | Verificar un puerto |

### Reportes
| Método | Ruta | Descripción |
|---|---|---|
| GET | `/api/Reports/recent-connections` | Conexiones recientes |
| GET | `/api/Reports/port-usage` | Uso de puertos |
| GET | `/api/Reports/device-history` | Historial por dispositivo |
| GET | `/api/Reports/port-activity` | Actividad por puerto |
| GET | `/api/Reports/connection-duration` | Duración de conexiones |
| GET | `/api/Reports/full-report` | Reporte completo |

## 🚀 Instalación

> Requisitos: Windows, **.NET 8 SDK**, **SQL Server** (LocalDB o instancia completa) y permisos de administrador.

1. **Restaurar y compilar** la solución:
   ```powershell
   dotnet restore "Proyecto Final.sln"
   dotnet build "Proyecto Final.sln" -c Release
   ```

2. **Crear la base de datos** ejecutando `MonitorPuertos.sql` en SQL Server (o usa `Configurar.ps1`).

3. **Ajustar la cadena de conexión** en `MonitorService/appsettings.json` y `MonitorAPI/appsettings.json` si tu servidor difiere de `localhost`.

4. **Instalar e iniciar el servicio** (requiere administrador):
   ```powershell
   .\Instalar.ps1
   .\Iniciar.ps1
   ```

5. **Abrir el dashboard** en el navegador (`https://localhost:62063`) o consultar la API con Swagger.

## 💻 Desarrollado por

**Richard Rodríguez** — [Death272004](https://github.com/Death272004)

Proyecto Final · Desarrollo de Software IV · Universidad Tecnológica de Panamá
