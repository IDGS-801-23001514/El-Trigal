# Documentación: Módulo de Categorías de Productos

## Descripción General
El módulo de Categorías de Productos es un CRUD completo que permite gestionar las categorías (clasificaciones) de productos de la panadería "El Trigal". Incluye funcionalidades de creación, lectura, actualización y eliminación (soft delete) de categorías.

## Estructura de Archivos

```
El-Trigal/
├── models.py                                  # Modelo CategoriaProducto
├── forms.py                                   # Formularios WTForms para categorías
├── api/categorias_productos/
│   ├── __init__.py                           # Exportar blueprint
│   └── routes.py                             # Rutas CRUD completas
└── templates/categorias-productos/
    ├── inicio.html                           # Listado con búsqueda y paginación
    ├── agregar.html                          # Formulario de creación
    ├── editar.html                           # Formulario de edición
    ├── detalle.html                          # Vista de detalles
    └── eliminar.html                         # Confirmación de desactivación
```

## Modelo Principal

### CategoriaProducto
```python
id(PK)                      # Identificador único
nombre                      # Nombre único de la categoría (3-65 caracteres)
descripcion                 # Descripción de la categoría (5-500 caracteres)
estatus                     # ACTIVO/INACTIVO (soft delete)
fecha_creacion              # Timestamp de creación
usuario_creacion(FK)        # Usuario que creó el registro
fecha_movimiento            # Último timestamp de cambio
usuario_movimiento(FK)      # Usuario que hizo el último cambio
```

## Formularios

### CategoriaProductoForm (Agregar)
- **nombre**: Validación de longitud (3-65), debe ser único
- **descripcion**: TextArea con validación (5-500 caracteres)
- Generador CSRF automático

### EditCategoriaProductoForm
Hereda de CategoriaProductoForm pero permite editar la misma categoría sin error de duplicado.

### BuscarCategoriaForm
Form simple para búsqueda por nombre.

### ConfirmarEliminacionCategoriaForm
Botones de confirmación (confirm/cancel) para soft delete.

## Validaciones

- **Nombre único** en la BD
- **Longitud nombre**: 3-65 caracteres
- **Longitud descripción**: 5-500 caracteres
- **Descripción requerida**
- **CSRF protection** en todos los formularios

## Rutas API

### 1. Listar Categorías
```
GET /categorias-productos/
GET /categorias-productos/inicio
```
**Parámetros Query:**
- `page` (int, default=1): Número de página
- `buscar` (str): Filtro por nombre (case-insensitive)

**Response:** HTML con categorías paginadas (12 por página)

### 2. Agregar Categoría
```
GET /categorias-productos/agregar      # Mostrar formulario
POST /categorias-productos/agregar     # Crear categoría
```
**Datos POST:**
- `nombre` (required, 3-65 caracteres)
- `descripcion` (required, 5-500 caracteres)
- CSRF token (automático)

**Response:**
- Éxito: Redirect a `/categorias-productos/` + flash "Categoría creada"
- Error: Re-render con errores

### 3. Ver Detalles
```
GET /categorias-productos/detalle/<id>
```
**Response:** Página con detalles completos de la categoría

### 4. Editar Categoría
```
GET /categorias-productos/editar/<id>    # Form pre-llenado
POST /categorias-productos/editar/<id>   # Actualizar
```
**Cambios Permitidos:**
- Nombre
- Descripción

### 5. Desactivar Categoría (Soft Delete)
```
GET /categorias-productos/eliminar/<id>     # Confirmación
POST /categorias-productos/eliminar/<id>    # Ejecutar desactivación
```
**Efecto:** Cambia `estatus` a 'INACTIVO' sin eliminar registros

## Características Implementadas

### Validaciones ✅
- Nombre único y rango 3-65 caracteres
- Descripción 5-500 caracteres, requerida
- CSRF protection
- Validación lado servidor

### Funcionalidades CRUD ✅
- **Create**: Agregar categorías
- **Read**: Listar y detalles
- **Update**: Editar información
- **Delete**: Soft delete (desactivar)

### UX/UI ✅
- Paginación (12 items por página)
- Búsqueda por nombre (ILIKE)
- Mensajes Flash (success/error/warning/info)
- Validación lado servidor con errores inline
- Formularios con estilos Tailwind

### Auditoría/Seguridad ✅
- Registro de usuario creación/modificación
- Timestamps automáticos
- Soft deletes (datos no se pierden)
- Estatus ACTIVO/INACTIVO por defecto
- CSRF protection en formularios

## Ejemplos de Uso

### Crear una Categoría
```bash
POST /categorias-productos/agregar HTTP/1.1
Content-Type: application/x-www-form-urlencoded

nombre=Panes%20Dulces&
descripcion=Categor%C3%ADa%20para%20panes%20dulces%20y%20postres&
csrf_token=<token>
```

### Buscar Categorías
```
GET /categorias-productos/?buscar=panes
```

### Editar Categoría
```bash
POST /categorias-productos/editar/1 HTTP/1.1
Content-Type: application/x-www-form-urlencoded

nombre=Panes%20Dulces&
descripcion=Panes%20y%20postres%20dulces%20variados&
csrf_token=<token>
```

## Manejo de Errores

### 404 - Categoría no encontrada
- Redirect a `/categorias-productos/` con mensaje "Categoría no existe"

### Categoría Inactiva
- Operaciones CRUD rechazadas
- Mensaje: "La categoría no está disponible / inactiva"

### Nombre Duplicado
- Validación rechaza nombres duplicados
- Mensaje: "Ya existe una categoría con este nombre"

### Error en Descripción
- Validación de rango (5-500 caracteres)
- Mensaje detallado del error

## Integración con Otros Módulos

### Productos
- Relación: `Producto.fk_categoria -> CategoriaProducto.id`
- Las categorías inactivas no aparecen en selects de productos
- No se puede eliminar categoría con productos asociados (considerar implementar)

## Configuración Requerida

### En `config.py`
```python
SQLALCHEMY_DATABASE_URI = "mysql+pymysql://root:1234@localhost/panaderia"
```

### En `app.py`
```python
from models import db
db.init_app(app)
```

### Dependencias
```
Flask-WTF==1.2.2
WTForms==3.2.1
SQLAlchemy==2.0.46
```

## Notas para Desarrollo

### TODO: Autenticación Real
```python
# En routes.py:
def get_user_id():
    # TODO: Implementar autenticación real
    return 1  # Por ahora retorna admin
```

### Restricción Recomendada
- Prevenir eliminación de categorías con productos asociados
- Mensajes informativos sobre productos dependientes

### Optimizaciones Futuras
- Caché de categorías activas para selects
- Reporte de uso de categorías
- Reordenamiento de categorías
