# TimeCheck — Sistema de Asistencia Corporativo

Sistema web de control de asistencia con **reconocimiento facial** en tiempo real.  
Los empleados se identifican con un ID de 7 dígitos y su cara; nunca se almacenan fotos, solo un vector numérico de 128 valores.

---

## Requisitos previos

Instala estas herramientas antes de continuar:

| Herramienta | Versión mínima | Descarga |
|---|---|---|
| Python | 3.10 o superior | https://www.python.org/downloads/ |
| MySQL | 8.0 o superior | https://dev.mysql.com/downloads/installer/ |
| Git | cualquiera | https://git-scm.com/downloads |
| Navegador | Chrome o Firefox | — |

> Durante la instalación de MySQL, cuando te pida contraseña de `root`, escribe **`root`** (o anótala, la necesitarás más adelante).

---

## Instalación paso a paso

### 1. Clonar el repositorio

```bash
git clone https://github.com/<tu-usuario>/<nombre-del-repo>.git
cd <nombre-del-repo>
```

---

### 2. Instalar dependencias de Python

```bash
pip install -r requirements.txt
```

Instala: Flask, flask-cors, PyMySQL y NumPy.

---

### 3. Configurar la base de datos

#### 3a. Conectarse a MySQL y ejecutar los scripts

Abre una terminal y conéctate a MySQL:

```bash
mysql -u root -p
```

Ingresa tu contraseña y luego ejecuta:

```sql
SOURCE asistencia_db.sql;
SOURCE biometria.sql;
```

> Si tu contraseña de MySQL **no es** `root`, abre `app.py`, busca `db_config` cerca de la línea 22 y cambia el valor de `'password'` por tu contraseña real.

#### 3b. Aplicar columnas adicionales

Estas columnas son necesarias para los módulos de Baja, Metadatos de acceso y Reportes:

```sql
USE sistema_asistencia;

ALTER TABLE empleados ADD COLUMN IF NOT EXISTS activo BOOLEAN DEFAULT TRUE;
UPDATE empleados SET activo = TRUE WHERE activo IS NULL;

ALTER TABLE asistencia ADD COLUMN IF NOT EXISTS puerta VARCHAR(60) NULL;
ALTER TABLE asistencia ADD COLUMN IF NOT EXISTS sucursal VARCHAR(60) NULL;

ALTER TABLE departamentos ADD COLUMN IF NOT EXISTS tolerancia_min INT DEFAULT 10;
UPDATE departamentos SET tolerancia_min = 10 WHERE tolerancia_min IS NULL;
```

---

### 4. Iniciar el servidor

```bash
python app.py
```

Deberías ver en la terminal:

```
 * Running on http://127.0.0.1:5000
 * Running on http://0.0.0.0:5000
```

---

### 5. Abrir la aplicación

Abre tu navegador y ve a:

```
http://localhost:5000
```

> **Importante:** Usa siempre `http://localhost:5000`. No abras `index.html` directamente como archivo porque el reconocimiento facial no funcionará.

---

## Estructura del proyecto

```
├── app.py                  # Backend Flask (API REST)
├── index.html              # Interfaz principal
├── script.js               # Lógica de navegación, empleados y reportes
├── biometria.js            # Módulo de reconocimiento facial (face-api.js)
├── style.css               # Estilos del panel de control
├── asistencia_db.sql       # Esquema base de datos + datos de prueba
├── biometria.sql           # Tabla de vectores faciales
├── requirements.txt        # Dependencias Python
└── models/                 # Modelos de IA para face-api.js (incluidos)
    ├── tiny_face_detector_model-*
    ├── face_landmark_68_model-*
    └── face_recognition_model-*
```

---

## Módulos del sistema

| Sección | Descripción |
|---|---|
| **Registro Asistencia** | El empleado ingresa su ID de 7 dígitos y verifica su rostro con la cámara |
| **Empleados** | Registrar nuevos empleados; el sistema genera un ID y abre la cámara para enrolar su cara |
| **Permisos e Incapacidades** | Registrar vacaciones, permisos y faltas por rango de fechas |
| **Reportes Diarios** | Ver entradas, salidas, retrasos, puerta y sucursal de todos los empleados del día |

---

## Primer uso recomendado

1. Ve a **Empleados** → llena nombre y departamento → haz clic en **Guardar Empleado**
2. El sistema genera un ID de 7 dígitos y abre la cámara automáticamente
3. Mira directo a la cámara hasta que aparezca el mensaje verde de éxito
4. Ve a **Registro Asistencia** → escribe ese ID → haz clic en **Registrar Entrada**
5. Verifica tu cara, selecciona puerta y sucursal, y confirma

---

## Problemas comunes

| Problema | Solución |
|---|---|
| `ModuleNotFoundError` al iniciar app.py | Corre `pip install -r requirements.txt` |
| `Access denied for user 'root'` | Cambia la contraseña en `app.py` → `db_config` |
| La cámara no abre | Asegúrate de acceder por `http://localhost:5000`, no por `file://` |
| Los modelos no cargan | Verifica que la carpeta `models/` exista dentro del proyecto clonado |
| `Unknown database 'sistema_asistencia'` | Ejecuta los archivos `.sql` en MySQL como se indica en el paso 3 |
