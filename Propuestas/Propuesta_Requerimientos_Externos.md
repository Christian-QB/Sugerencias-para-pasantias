# Propuesta técnica y funcional: Módulo de Requerimientos Externos

## 1. Propósito del documento

Este documento presenta la propuesta inicial para el diseño de un nuevo módulo denominado **Requerimientos Externos**. El objetivo es organizar, centralizar y controlar los requerimientos reportados por empresas externas que utilizan el sistema web, sin necesidad de ingresar directamente a la plataforma o base de datos de cada cliente.

---

## 2. Contexto actual del sistema

Actualmente el sistema web es utilizado por empresas que gestionan procesos como ventas, inventario, almacenamiento, reportes, usuarios, proyectos y otros módulos operativos. Dentro del sistema ya existe un módulo de **Ticket Soporte / Requerimientos**, el cual permite registrar solicitudes internas dentro de la empresa donde el usuario inició sesión.

En el entorno actual se identifican dos empresas principales con acceso directo a sus bases de datos:

```text
Empresas principales con acceso directo
├── DiscomRaiz
└── OnlyNatural
```

Para fines de análisis, se considera a **DiscomRaiz** como la empresa matriz principal, debido a que es el entorno donde se ha trabajado directamente durante la pasantía. **OnlyNatural** también forma parte del ecosistema principal, aunque el acceso y flujo exacto de operación puede variar.

El sistema, sin embargo, no está pensado únicamente para estas dos empresas. Es un sistema web distribuible que puede ser vendido, configurado o utilizado por otras empresas según sus necesidades. Por esta razón, el soporte técnico no debe depender exclusivamente de tener acceso directo a la base de datos o a la sesión administrativa de cada empresa cliente.

---

## 3. Problema identificado

El módulo actual de requerimientos funciona dentro de la empresa donde el usuario ha iniciado sesión. Por ejemplo, si un usuario de DiscomRaiz registra un requerimiento, este queda almacenado en la base de datos de DiscomRaiz. Si un administrador de OnlyNatural ingresa a su entorno, consulta los requerimientos propios de OnlyNatural.

Esta lógica es funcional para empresas con acceso directo. Sin embargo, se presenta una limitación cuando una empresa externa reporta un problema y el equipo de soporte no tiene acceso directo a su plataforma o base de datos.

En ese caso, el soporte necesita registrar el requerimiento de forma centralizada, desde su propio entorno, sin depender de iniciar sesión en la empresa cliente.

El problema se puede resumir así:

```text
El sistema actual registra requerimientos dentro de cada empresa.
El nuevo módulo debe permitir registrar requerimientos externos desde una base central propia.
```

---

## 4. Objetivo general del nuevo módulo

Diseñar e implementar un módulo llamado **Requerimientos Externos**, orientado a registrar, consultar, actualizar, clasificar y reportar requerimientos de empresas externas que utilizan el sistema, manteniendo una administración centralizada desde el área de soporte.

Este módulo permitirá:

- registrar empresas externas de forma dinámica;
- registrar usuarios asociados a empresas externas;
- registrar requerimientos sin iniciar sesión en la plataforma del cliente;
- asociar cada requerimiento a una empresa y a un usuario específico;
- mantener historial de problemas por empresa y por usuario;
- clasificar si el requerimiento corresponde a un problema del sistema o a un problema de uso del usuario;
- identificar requerimientos facturables y no facturables;
- generar reportes más claros para soporte, jefatura y posible facturación;
- preparar el sistema para una futura centralización automática de requerimientos.

---

## 5. Principios de diseño

El nuevo módulo debe respetar los siguientes principios:

### 5.1 Dinamismo

Toda la información debe ser administrable desde el sistema. No se deben quemar empresas, usuarios, módulos, tipos de requerimientos, causas o estados directamente en el código.

No se recomienda una lógica de este tipo:

```php
if ($empresa == 'DiscomRaiz') {
    // lógica específica
}
```

La lógica correcta debe basarse en catálogos y tablas administrables.

### 5.2 Separación del módulo actual

El módulo **Ticket Soporte / Requerimientos** actual debe mantenerse separado del nuevo módulo **Requerimientos Externos**. Esto evita afectar un flujo que ya funciona y permite desarrollar la nueva lógica de forma limpia.

La separación propuesta es:

```text
Ticket Soporte / Requerimientos
→ Requerimientos internos de la empresa donde se inició sesión.

Requerimientos Externos
→ Requerimientos centralizados de empresas externas o clientes del sistema.
```

### 5.3 Modularidad

La lógica debe dividirse en archivos organizados por responsabilidad:

- vistas;
- procesos;
- consultas reutilizables;
- validaciones;
- catálogos;
- reportes;
- utilidades.

### 5.4 Escalabilidad

El módulo debe permitir agregar nuevas empresas y usuarios sin modificar el código fuente. También debe estar preparado para una posible integración futura mediante API o sincronización automática.

### 5.5 Compatibilidad con PHP 7

La implementación debe ser compatible con **PHP 7**, evitando características exclusivas de PHP 8 o superior. Se recomienda:

- usar `mysqli` o `PDO` de forma consistente según el estándar del proyecto;
- utilizar consultas preparadas;
- evitar typed properties de PHP 7.4+ si el servidor no está confirmado;
- evitar atributos de PHP 8;
- mantener una sintaxis compatible con el entorno actual;
- documentar funciones con PHPDoc cuando sea necesario.

---

## 6. Alcance inicial del módulo

El alcance inicial del módulo **Requerimientos Externos** incluiría:

```text
Requerimientos Externos
├── Empresas externas
├── Usuarios externos
├── Registro de requerimiento externo
├── Actualización de requerimiento externo
├── Clasificación del requerimiento
├── Seguimiento / bitácora
├── Reporte por empresa
├── Reporte por usuario
├── Reporte facturable / no facturable
└── Exportación a Excel
```

No se plantea reemplazar el módulo actual de Ticket Soporte. La propuesta es crear una capa adicional para requerimientos externos.

---

## 7. Entidades principales propuestas

### 7.1 Empresas externas

Se requiere una tabla para registrar las empresas clientes que usan el sistema o reciben soporte.

Nombre sugerido de tabla:

```text
tb_ext_empresas
```

Campos sugeridos:

```text
id_empresa
ruc
nombre_comercial
razon_social
telefono
correo
direccion
estado
observacion
fecha_registro
usuario_registro
fecha_modificacion
usuario_modificacion
```

#### Consideraciones

- El **RUC** debería ser el identificador principal de la empresa.
- El nombre de la empresa no debe ser el identificador único, porque puede variar o repetirse.
- La empresa debe poder activarse o desactivarse sin eliminar historial.
- DiscomRaiz y OnlyNatural podrían registrarse también como empresas dentro de este catálogo central para fines de reportes unificados.

---

### 7.2 Usuarios externos

Se requiere registrar usuarios de empresas externas para asociarlos a los requerimientos.

Nombre sugerido de tabla:

```text
tb_ext_usuarios
```

Campos sugeridos:

```text
id_usuario_externo
id_empresa
tipo_documento
numero_documento
nombres
apellidos
telefono
correo
cargo
departamento
estado
fecha_registro
usuario_registro
fecha_modificacion
usuario_modificacion
```

#### Consideraciones

Aunque inicialmente se pensó en utilizar la cédula como identificador principal, se recomienda manejarlo de forma más flexible mediante:

```text
tipo_documento + numero_documento
```

Esto permite soportar:

- cédula;
- RUC;
- pasaporte;
- documento extranjero;
- identificadores especiales según la empresa.

El sistema debe evitar registros duplicados de usuarios dentro de una misma empresa.

---

### 7.3 Requerimientos externos

Esta sería la tabla central del nuevo módulo.

Nombre sugerido de tabla:

```text
tb_ext_requerimientos
```

Campos sugeridos:

```text
id_requerimiento
codigo_ticket
id_empresa
id_usuario_externo
id_modulo
id_tipo_requerimiento
id_prioridad
descripcion
origen_contacto
estado
responsable_soporte
clasificacion_inicial
clasificacion_final
facturable
motivo_facturacion
solucion
fecha_registro
fecha_tramite
fecha_conclusion
fecha_aprobacion
usuario_registro
usuario_modificacion
fecha_modificacion
```

#### Consideraciones

El requerimiento debe estar asociado obligatoriamente a:

- una empresa;
- un usuario externo;
- un módulo o área afectada;
- una descripción;
- un estado;
- una prioridad.

El código del ticket debe generarse automáticamente para facilitar el seguimiento.

---

### 7.4 Módulos del sistema

Para mantener todo dinámico, los módulos deben manejarse como catálogo.

Nombre sugerido de tabla:

```text
tb_ext_modulos
```

Campos sugeridos:

```text
id_modulo
nombre_modulo
descripcion
estado
fecha_registro
usuario_registro
```

Ejemplos:

```text
Ventas
Inventario
Reportes
Usuarios
Contabilidad
Facturación
Tributación
Proyectos
```

---

### 7.5 Tipos de requerimiento

Nombre sugerido de tabla:

```text
tb_ext_tipos_requerimiento
```

Campos sugeridos:

```text
id_tipo_requerimiento
nombre_tipo
descripcion
estado
fecha_registro
usuario_registro
```

Ejemplos:

```text
Error del sistema
Mejora solicitada
Soporte de uso
Configuración
Capacitación
Consulta funcional
Problema de acceso
```

---

### 7.6 Prioridades

Nombre sugerido de tabla:

```text
tb_ext_prioridades
```

Campos sugeridos:

```text
id_prioridad
nombre_prioridad
descripcion
nivel
estado
```

Ejemplos:

```text
Urgencia Alta
Urgencia Media
Urgencia Baja
```

---

### 7.7 Clasificación de responsabilidad

Esta tabla permitirá diferenciar si el requerimiento corresponde a un problema del sistema o a una situación causada por el usuario, la empresa o un factor externo.

Nombre sugerido de tabla:

```text
tb_ext_clasificaciones
```

Campos sugeridos:

```text
id_clasificacion
nombre_clasificacion
descripcion
facturable_sugerido
estado
```

Ejemplos:

```text
Sistema / proveedor
Uso incorrecto del usuario
Falta de capacitación
Configuración solicitada por cliente
Error de datos ingresados por cliente
Infraestructura del cliente
Mejora no incluida en contrato
Consulta funcional
Pendiente de análisis
```

#### Recomendación de lenguaje

No se recomienda utilizar una etiqueta como “culpa del usuario” dentro del sistema. Es mejor utilizar términos profesionales como:

```text
Uso incorrecto del usuario
Falta de capacitación
Error de datos ingresados por cliente
```

Esto permite generar reportes más formales y evita conflictos de comunicación con el cliente.

---

### 7.8 Seguimiento o bitácora

Cada requerimiento debería tener historial de acciones.

Nombre sugerido de tabla:

```text
tb_ext_requerimiento_seguimiento
```

Campos sugeridos:

```text
id_seguimiento
id_requerimiento
estado_anterior
estado_nuevo
comentario
usuario_responsable
fecha_registro
```

#### Utilidad

Permite conocer:

- quién registró el requerimiento;
- quién lo tomó;
- qué acciones se hicieron;
- cuándo cambió de estado;
- por qué se clasificó como facturable o no facturable;
- cuál fue la solución aplicada.

---

### 7.9 Adjuntos y evidencias

Nombre sugerido de tabla:

```text
tb_ext_requerimiento_adjuntos
```

Campos sugeridos:

```text
id_adjunto
id_requerimiento
nombre_archivo
tipo_archivo
ruta_archivo_o_blob
fecha_subida
usuario_subida
estado
```

#### Tipos de evidencia

- capturas de pantalla;
- documentos PDF;
- imágenes;
- archivos de soporte;
- evidencia del error reportado.

---

## 8. Clasificación inicial y clasificación final

No siempre es posible saber desde el registro si el problema es responsabilidad del sistema o del usuario. Por eso se recomienda manejar dos momentos de clasificación.

### 8.1 Clasificación inicial

Se define al momento de registrar el requerimiento. Puede quedar como:

```text
Pendiente de análisis
```

o puede seleccionarse una categoría inicial si el caso es evidente.

### 8.2 Clasificación final

Se define al cerrar el requerimiento, luego de revisar el caso.

Ejemplos:

```text
Sistema / proveedor
Uso incorrecto del usuario
Falta de capacitación
Infraestructura del cliente
Mejora no incluida en contrato
```

Esta clasificación final será la base para reportes y posibles cobros.

---

## 9. Lógica facturable / no facturable

El módulo debe permitir clasificar si un requerimiento será facturable o no.

Campos propuestos:

```text
facturable
motivo_facturacion
```

Valores sugeridos para `facturable`:

```text
Sí
No
Pendiente
```

### Casos no facturables

Ejemplos:

- error real del sistema;
- falla cubierta por contrato;
- incidencia generada por actualización del proveedor;
- error técnico atribuible al sistema distribuido.

### Casos facturables

Ejemplos:

- capacitación adicional;
- uso incorrecto reiterado;
- problema causado por datos mal ingresados;
- configuración especial solicitada por el cliente;
- mejora fuera del alcance contratado;
- soporte que no corresponde a fallas del sistema.

### Utilidad

Esta lógica permitirá generar reportes como:

```text
Empresa: Cliente A
Total de requerimientos: 25
No facturables: 15
Facturables: 10
Motivo principal: falta de capacitación del usuario
```

---

## 10. Flujo propuesto: registro de requerimiento externo

El flujo inicial sería:

```text
1. Soporte recibe llamada, WhatsApp, correo o mensaje de una empresa externa.
2. Soporte ingresa al módulo Requerimientos Externos.
3. Busca la empresa por RUC o nombre.
4. Si la empresa no existe, la registra.
5. Busca al usuario por cédula o documento.
6. Si el usuario no existe, lo registra y lo asocia a la empresa.
7. Crea el requerimiento.
8. Selecciona módulo afectado.
9. Selecciona tipo de requerimiento.
10. Define prioridad.
11. Escribe descripción del problema.
12. Adjunta evidencia si existe.
13. Define origen del contacto.
14. Deja clasificación inicial como Pendiente de análisis, si aún no se conoce la causa.
15. Guarda el ticket.
16. El sistema genera un código de requerimiento.
```

---

## 11. Flujo propuesto: actualización y cierre

```text
1. Soporte revisa el requerimiento.
2. Cambia estado a Trámite o En análisis.
3. Registra observaciones en la bitácora.
4. Determina la causa real del problema.
5. Define clasificación final.
6. Marca si es facturable o no facturable.
7. Registra la solución aplicada.
8. Adjunta evidencia de solución, si aplica.
9. Cambia estado a Concluido.
10. El requerimiento puede aprobarse o cerrarse según la lógica definida.
```

---

## 12. Estados recomendados

Para mantener compatibilidad con la lógica actual del módulo de requerimientos, se recomienda iniciar con estados similares:

```text
Pendiente
Trámite
Concluido
Aprobado
```

Como mejora futura, se podría ampliar el flujo a:

```text
Pendiente
En análisis
En trámite
Esperando cliente
Concluido
Aprobado
Rechazado
Cancelado
```

### Recomendación inicial

Para la primera versión del módulo, se recomienda mantener la menor cantidad de estados posible y no complicar el flujo antes de validar el proceso con el jefe del área.

---

## 13. Reportes propuestos

### 13.1 Reporte por empresa

Debe permitir consultar:

```text
Empresa
Total de requerimientos
Pendientes
En trámite
Concluidos
Aprobados
Facturables
No facturables
```

### 13.2 Reporte por usuario

Debe permitir consultar:

```text
Usuario
Empresa
Cantidad de requerimientos
Problemas del sistema
Problemas por uso incorrecto
Último requerimiento registrado
```

### 13.3 Reporte facturable / no facturable

Debe permitir consultar:

```text
Empresa
Mes
Cantidad de requerimientos facturables
Motivo
Tiempo invertido
Valor sugerido, si se define una tarifa
```

### 13.4 Reporte de capacitación

Debe permitir detectar:

```text
Usuarios con más requerimientos por uso incorrecto
Empresas con más casos por falta de capacitación
Módulos con más errores de uso
```

Esto serviría para sugerir capacitaciones al cliente.

### 13.5 Exportación a Excel

Todos los reportes principales deberían poder exportarse a Excel o CSV para revisión offline.

---

## 14. Pantallas sugeridas

### 14.1 Gestión de empresas externas

Funcionalidades:

```text
Registrar empresa
Editar empresa
Activar/desactivar empresa
Buscar empresa por RUC o nombre
Ver historial de requerimientos por empresa
```

### 14.2 Gestión de usuarios externos

Funcionalidades:

```text
Registrar usuario
Editar usuario
Asociar usuario a empresa
Buscar usuario por documento
Ver historial del usuario
```

### 14.3 Registrar requerimiento externo

Campos sugeridos:

```text
Empresa
Usuario externo
Módulo afectado
Tipo de requerimiento
Prioridad
Origen del contacto
Descripción
Adjunto
Clasificación inicial
```

### 14.4 Actualizar requerimiento externo

Campos sugeridos:

```text
Estado
Responsable
Comentario de seguimiento
Clasificación final
Facturable
Motivo de facturación
Solución aplicada
Adjunto de evidencia
```

### 14.5 Reportes

Funcionalidades:

```text
Filtrar por empresa
Filtrar por usuario
Filtrar por estado
Filtrar por clasificación
Filtrar por facturable/no facturable
Exportar a Excel
```

---

## 15. Futuro: centralización automática

A futuro, el módulo podría evolucionar para recibir requerimientos directamente desde los sistemas instalados en cada empresa cliente.

La lógica futura sería:

```text
Sistema del cliente
→ usuario registra problema relacionado con el sistema web
→ el sistema envía el requerimiento al servidor central de soporte
→ soporte lo recibe automáticamente en Requerimientos Externos
```

Para esto se podría implementar una API central.

Elementos necesarios:

```text
endpoint central de recepción
api_key por empresa
token de autenticación
validación de empresa activa
registro automático de ticket
sincronización de estado, si aplica
```

Esta fase no forma parte del alcance inicial, pero el diseño del módulo debe estar preparado para crecer hacia esa integración.

---

## 16. Consideraciones técnicas para PHP 7

El desarrollo debe considerar compatibilidad con PHP 7.

Recomendaciones:

```text
Usar require_once para archivos comunes.
Evitar características exclusivas de PHP 8.
Usar consultas preparadas con mysqli o PDO.
Validar todos los datos recibidos por POST y GET.
Aplicar htmlspecialchars al mostrar datos ingresados por usuarios.
No aplicar htmlspecialchars antes de guardar en base de datos.
Usar utf8mb4 para conexión y almacenamiento de caracteres especiales.
Separar lógica PHP, vistas, procesos y funciones reutilizables.
Documentar funciones con PHPDoc si no se usan tipos estrictos.
```

---

## 17. Seguridad y validaciones

El módulo debe contemplar:

```text
Validación de sesión interna.
Validación de permisos según usuario interno.
Validación de empresa activa.
Validación de usuario externo activo.
Validación de campos obligatorios.
Validación de archivos adjuntos.
Control de tamaño y tipo de archivo.
Prevención de duplicados de empresas y usuarios.
Protección contra inyección SQL mediante consultas preparadas.
Protección XSS mediante htmlspecialchars en salida HTML.
```

---

## 18. Estructura técnica propuesta del módulo

Estructura sugerida:

```text
vista/pages/requerimientos_externos/
├── empresasExternas.php
├── empresaExternaProceso.php
├── usuariosExternos.php
├── usuarioExternoProceso.php
├── registrarRequerimientoExterno.php
├── requerimientoExternoProceso.php
├── actualizarRequerimientoExterno.php
├── actualizarRequerimientoExternoProceso.php
├── reporteRequerimientosExternos.php
├── exportarRequerimientosExternos.php
├── dashboardRequerimientosExternos.php
├── includes/
│   ├── authExternos.php
│   ├── empresaQueries.php
│   ├── usuarioQueries.php
│   ├── requerimientoExternoQueries.php
│   ├── seguimientoQueries.php
│   ├── catalogosExternos.php
│   ├── validacionesExternas.php
│   ├── estadosExternos.php
│   ├── clasificacionesExternas.php
│   └── notificacionesExternas.php
└── assets/
    ├── requerimientosExternos.js
    └── requerimientosExternos.css
```

---

## 19. Descripción de archivos propuestos

### `empresasExternas.php`

Pantalla para listar, buscar, registrar y editar empresas externas.

### `empresaExternaProceso.php`

Archivo de proceso para guardar o actualizar empresas externas.

### `usuariosExternos.php`

Pantalla para listar, buscar, registrar y editar usuarios externos asociados a empresas.

### `usuarioExternoProceso.php`

Archivo de proceso para guardar o actualizar usuarios externos.

### `registrarRequerimientoExterno.php`

Formulario principal para registrar un requerimiento externo.

### `requerimientoExternoProceso.php`

Proceso encargado de validar y guardar el nuevo requerimiento externo.

### `actualizarRequerimientoExterno.php`

Pantalla para revisar, actualizar, clasificar y cerrar requerimientos externos.

### `actualizarRequerimientoExternoProceso.php`

Archivo encargado de procesar cambios de estado, clasificación final, facturación y solución.

### `reporteRequerimientosExternos.php`

Reporte general del módulo, con filtros por empresa, usuario, estado, clasificación y facturación.

### `exportarRequerimientosExternos.php`

Archivo para exportar reportes a Excel o CSV.

### `dashboardRequerimientosExternos.php`

Panel visual con indicadores generales del soporte externo.

### `includes/authExternos.php`

Validación de sesión y acceso al módulo.

### `includes/empresaQueries.php`

Funciones reutilizables para consultar, crear o actualizar empresas.

### `includes/usuarioQueries.php`

Funciones reutilizables para consultar, crear o actualizar usuarios externos.

### `includes/requerimientoExternoQueries.php`

Funciones principales para registrar, consultar y actualizar requerimientos externos.

### `includes/seguimientoQueries.php`

Funciones relacionadas con la bitácora del requerimiento.

### `includes/catalogosExternos.php`

Carga dinámica de módulos, tipos, prioridades y clasificaciones.

### `includes/validacionesExternas.php`

Validaciones reutilizables de empresa, usuario, requerimiento, archivos y estados.

### `includes/estadosExternos.php`

Reglas de cambio de estado.

### `includes/clasificacionesExternas.php`

Reglas relacionadas con responsabilidad y facturación.

### `includes/notificacionesExternas.php`

Funciones para notificar por correo o WhatsApp si se decide implementar.

### `assets/requerimientosExternos.js`

JavaScript propio del módulo.

### `assets/requerimientosExternos.css`

Estilos propios del módulo.

---

## 20. Fases de implementación recomendadas

### Fase 1: Validación de la propuesta

```text
Revisar alcance con el Ing Villalva.
Confirmar tablas necesarias.
Confirmar campos obligatorios.
Confirmar si se requiere facturación desde la primera versión.
Confirmar reportes prioritarios.
```

### Fase 2: Catálogo de empresas externas

```text
Crear tabla de empresas.
Crear pantalla de registro y edición.
Validar RUC único.
Permitir activar/desactivar empresas.
```

### Fase 3: Catálogo de usuarios externos

```text
Crear tabla de usuarios externos.
Asociar usuarios a empresas.
Validar documento único por empresa.
Permitir consulta de historial por usuario.
```

### Fase 4: Registro de requerimiento externo

```text
Crear formulario de requerimiento externo.
Seleccionar empresa y usuario.
Registrar descripción, módulo, prioridad y evidencia.
Generar código de ticket.
```

### Fase 5: Actualización y clasificación

```text
Actualizar estado.
Registrar seguimiento.
Definir clasificación final.
Marcar facturable o no facturable.
Registrar solución.
```

### Fase 6: Reportes y exportación

```text
Reporte por empresa.
Reporte por usuario.
Reporte facturable/no facturable.
Exportación a Excel.
```

### Fase 7: Centralización automática futura

```text
Diseñar endpoint central.
Permitir que sistemas cliente envíen tickets.
Validar empresas por API key.
Sincronizar información de soporte.
```

---

## 21. Decisiones pendientes para revisión con jefatura

Antes de desarrollar, porfavor confirmar:

```text
¿El módulo será exclusivo para el área de sistemas?
¿Se registrarán DiscomRaiz y OnlyNatural también como empresas externas/cliente?
¿El RUC será obligatorio para toda empresa?
¿La cédula será obligatoria para todo usuario externo?
¿Se permitirá registrar usuarios sin documento en casos especiales?
¿Qué estados se usarán en la primera versión?
¿Qué clasificaciones serán obligatorias?
¿La opción facturable se definirá al registrar o al cerrar?
¿Se manejará tarifa o solo reporte de casos facturables?
¿Se enviarán notificaciones al cliente?
¿Se exportará inicialmente como CSV o XLSX?
¿La centralización automática será una fase futura o debe considerarse desde el inicio?
```

---

## 22. Conclusión

El módulo **Requerimientos Externos** permitiría al área de sistemas registrar y controlar requerimientos de empresas externas desde una base central propia, sin depender de iniciar sesión en la plataforma de cada cliente.

La propuesta mantiene separado el módulo actual de **Ticket Soporte / Requerimientos**, evitando afectar el flujo existente. Además, plantea una estructura dinámica, modular y escalable para gestionar empresas, usuarios, tickets, clasificaciones, reportes y posibles cobros por soporte no atribuible al sistema.

El módulo permitiría mejorar la organización del soporte, identificar patrones de problemas por empresa o usuario, justificar capacitaciones, diferenciar incidencias reales del sistema frente a problemas de uso, y preparar el sistema para una futura centralización automática de requerimientos.

Esta propuesta queda sujeta a revisión y aprobación del jefe del área de sistemas antes de iniciar el desarrollo.
