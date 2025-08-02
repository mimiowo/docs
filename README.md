# Especificación de Requerimientos de Software (IEEE 830)
## Sistema Co-Link - Portal de Cooperativa de Vivienda

### 1. INTRODUCCIÓN

#### 1.1 Propósito
Este documento establece los requerimientos funcionales y no funcionales para el desarrollo del sistema Co-Link, una plataforma web integral para la gestión de cooperativas de vivienda que facilita la administración de socios, pagos, registro de horas de trabajo y comunicación entre los miembros.

#### 1.2 Alcance
El sistema Co-Link permitirá:
- Gestión de usuarios y perfiles de socios
- Registro y seguimiento de horas de trabajo
- Administración de pagos y cuotas
- Portal administrativo para gestión de la cooperativa
- Sistema de autenticación y autorización
- Notificaciones y comunicaciones

#### 1.3 Definiciones y Acrónimos
- **Co-Link**: Sistema de gestión de cooperativa de vivienda
- **Socio**: Miembro de la cooperativa
- **Admin**: Administrador del sistema
- **API**: Interfaz de Programación de Aplicaciones
- **UI**: Interfaz de Usuario

### 2. DESCRIPCIÓN GENERAL

#### 2.1 Perspectiva del Producto
Co-Link es un sistema web independiente compuesto por:
- Frontend web responsive
- API REST backend
- Base de datos relacional
- Sistema de autenticación OAuth2

#### 2.2 Funciones del Producto
- **RF001**: Registro de nuevos socios
- **RF002**: Autenticación de usuarios
- **RF003**: Gestión de perfiles de usuario
- **RF004**: Registro de horas de trabajo
- **RF005**: Subida y gestión de comprobantes de pago
- **RF006**: Dashboard administrativo
- **RF007**: Aprobación/rechazo de solicitudes
- **RF008**: Generación de reportes

#### 2.3 Características de los Usuarios
- **Socios**: Usuarios finales que registran horas y pagos
- **Administradores**: Gestionan la cooperativa y aprueban solicitudes

### 3. REQUERIMIENTOS ESPECÍFICOS

#### 3.1 Requerimientos Funcionales

| ID | Descripción | Prioridad | Entrada | Proceso | Salida |
|----|-------------|-----------|---------|---------|--------|
| RF001 | Registro de Usuario | Alta | Datos personales, email, cédula | Validación y almacenamiento | Confirmación de registro |
| RF002 | Login de Usuario | Alta | Email y contraseña | Autenticación OAuth2 | Token de acceso |
| RF003 | Gestión de Perfil | Media | Datos de usuario | Actualización de datos | Confirmación de cambios |
| RF004 | Registro de Horas | Alta | Fecha, horas, descripción | Validación y almacenamiento | Registro guardado |
| RF005 | Subida de Pagos | Alta | Archivo, monto, fecha | Procesamiento de archivo | Comprobante almacenado |
| RF006 | Dashboard Admin | Alta | Solicitud de datos | Consulta a BD | Estadísticas y listados |
| RF007 | Aprobación de Usuarios | Alta | ID usuario, decisión | Actualización de estado | Notificación al usuario |
| RF008 | Reportes | Media | Filtros de búsqueda | Procesamiento de datos | Reporte generado |

#### 3.2 Requerimientos No Funcionales

| ID | Categoría | Descripción | Métrica |
|----|-----------|-------------|---------|
| RNF001 | Rendimiento | Tiempo de respuesta API | < 2 segundos |
| RNF002 | Usabilidad | Interfaz responsive | Funcional en móviles/tablets |
| RNF003 | Seguridad | Autenticación OAuth2 | Tokens JWT |
| RNF004 | Disponibilidad | Uptime del sistema | 99.5% |
| RNF005 | Escalabilidad | Usuarios concurrent | Hasta 100 usuarios |
| RNF006 | Compatibilidad | Navegadores soportados | Chrome, Firefox, Safari, Edge |

### 4. MODELO DE DATOS NORMALIZADO (3FN)

#### 4.1 Tabla: usuarios
| Campo | Tipo | Restricciones |
|-------|------|---------------|
| id | INT | PK, AUTO_INCREMENT |
| name | VARCHAR(255) | NOT NULL |
| email | VARCHAR(255) | UNIQUE, NOT NULL |
| password | VARCHAR(255) | NOT NULL |
| ci | VARCHAR(20) | UNIQUE, NOT NULL |
| telefono | VARCHAR(20) | NULL |
| direccion | VARCHAR(255) | NULL |
| fecha_nacimiento | DATE | NULL |
| ingresos | DECIMAL(10,2) | NULL |
| motivacion | TEXT | NULL |
| aprobado | BOOLEAN | DEFAULT FALSE |
| is_admin | BOOLEAN | DEFAULT FALSE |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP |
| updated_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP |

#### 4.2 Tabla: registro_horas
| Campo | Tipo | Restricciones |
|-------|------|---------------|
| id | INT | PK, AUTO_INCREMENT |
| user_id | INT | FK(usuarios.id), NOT NULL |
| etapa_obra_id | INT | FK(etapas_obra.id), NOT NULL |
| fecha | DATE | NOT NULL |
| horas | DECIMAL(4,2) | NOT NULL |
| descripcion | TEXT | NULL |
| aprobado | BOOLEAN | DEFAULT FALSE |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP |

#### 4.3 Tabla: registro_pagos
| Campo | Tipo | Restricciones |
|-------|------|---------------|
| id | INT | PK, AUTO_INCREMENT |
| user_id | INT | FK(usuarios.id), NOT NULL |
| monto | DECIMAL(10,2) | NOT NULL |
| fecha_pago | DATE | NOT NULL |
| comprobante_path | VARCHAR(500) | NULL |
| aprobado | BOOLEAN | DEFAULT FALSE |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP |

#### 4.4 Tabla: etapas_obra
| Campo | Tipo | Restricciones |
|-------|------|---------------|
| id | INT | PK, AUTO_INCREMENT |
| nombre | VARCHAR(255) | NOT NULL |
| descripcion | TEXT | NULL |
| activa | BOOLEAN | DEFAULT TRUE |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP |

#### 4.5 Tabla: unidad_habitacional
| Campo | Tipo | Restricciones |
|-------|------|---------------|
| id | INT | PK, AUTO_INCREMENT |
| user_id | INT | FK(usuarios.id), NOT NULL |
| numero_unidad | VARCHAR(10) | NOT NULL |
| tipo | VARCHAR(50) | NOT NULL |
| metros_cuadrados | DECIMAL(6,2) | NOT NULL |
| asignada | BOOLEAN | DEFAULT FALSE |
| created_at | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP |

### 5. DIAGRAMA ENTIDAD-RELACIÓN

```
[USUARIOS] ||--o{ [REGISTRO_HORAS] : registra
[USUARIOS] ||--o{ [REGISTRO_PAGOS] : realiza
[USUARIOS] ||--o| [UNIDAD_HABITACIONAL] : posee
[ETAPAS_OBRA] ||--o{ [REGISTRO_HORAS] : clasifica

USUARIOS (1:N) REGISTRO_HORAS
USUARIOS (1:N) REGISTRO_PAGOS  
USUARIOS (1:1) UNIDAD_HABITACIONAL
ETAPAS_OBRA (1:N) REGISTRO_HORAS
```

### 6. REGLAS DE NEGOCIO ESPECÍFICAS (RNE)

| ID | Descripción | Tabla Afectada |
|----|-------------|----------------|
| RNE001 | Un usuario solo puede registrar máximo 8 horas por día | registro_horas |
| RNE002 | Los pagos deben ser aprobados por un administrador | registro_pagos |
| RNE003 | Solo usuarios aprobados pueden acceder al portal | usuarios |
| RNE004 | La cédula debe ser única en el sistema | usuarios |
| RNE005 | Las horas de trabajo deben estar asociadas a una etapa activa | registro_horas |
| RNE006 | Los archivos de comprobantes deben ser PDF o imágenes | registro_pagos |
| RNE007 | Solo un administrador puede asignar unidades habitacionales | unidad_habitacional |

### 7. CASOS DE USO PRINCIPALES

#### CU001: Registro de Usuario
**Actor**: Socio  
**Precondición**: Tener datos personales válidos  
**Flujo Principal**:
1. Usuario accede al formulario de registro
2. Completa datos personales obligatorios
3. Sistema valida información
4. Sistema guarda solicitud como "pendiente"
5. Administrador recibe notificación

#### CU002: Registro de Horas
**Actor**: Socio autenticado  
**Precondición**: Usuario aprobado y autenticado  
**Flujo Principal**:
1. Usuario accede al portal
2. Selecciona "Registrar Horas"
3. Completa formulario (fecha, horas, etapa)
4. Sistema valida reglas de negocio
5. Registro queda pendiente de aprobación

#### CU003: Gestión Administrativa
**Actor**: Administrador  
**Precondición**: Usuario con permisos de admin  
**Flujo Principal**:
1. Admin accede al backoffice
2. Visualiza dashboard con estadísticas
3. Revisa solicitudes pendientes
4. Aprueba/rechaza según criterios
5. Sistema notifica decisiones
