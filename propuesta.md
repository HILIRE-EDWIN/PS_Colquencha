# Plataforma de atención ciudadana con base de datos para el Portal Web del Gobierno Autónomo Municipal de Colquencha

**Propuesta de Proyecto Integrador**
Programa Académico Desconcentrado "Técnico Superior en Informática"
UMSA · Sede Colquencha

Autor: Edwin Gerardo Hilire Blanco
Fecha: 17 de noviembre de 2026

---

## Resumen

El Portal Web del Gobierno Autónomo Municipal de Colquencha (GAMC), construido en el módulo de Programación Web, permite al vecino consultar trámites, requisitos, aranceles y horarios, organizar su carpeta de documentos y ubicar la Alcaldía. Sin embargo, toda su información vive en archivos JavaScript estáticos y no existe un servidor que reciba y conserve las solicitudes ciudadanas. Cambiar un arancel exige modificar código, y los tickets de atención se generan en el navegador de cada persona, sin un correlativo único ni un registro que el municipio pueda consultar.

Este proyecto integrador agrega al portal la capa que le falta: una base de datos relacional MySQL, un servicio en PHP que entrega el catálogo de trámites desde la base de datos, el registro de solicitudes de atención ciudadana con ticket correlativo único, la consulta de estado por parte del ciudadano y un panel básico para que el personal municipal administre trámites y solicitudes. El producto se verifica con una matriz de casos de prueba y se pone en producción sobre hosting compartido. El desarrollo se planifica en cuatro semanas, del 23 de noviembre al 18 de diciembre de 2026.

**Palabras clave:** gobierno municipal, atención ciudadana, trámites, base de datos relacional, PHP, MySQL, Colquencha.

---

## Introducción

El municipio de Colquencha, en la provincia Aroma del altiplano paceño, agrupa comunidades y ayllus dispersos a distancias considerables del centro administrativo de la Plaza 28 de Agosto. Históricamente, esa distancia obligaba a muchos comunarios a viajar solo para averiguar requisitos, aranceles u horarios de atención. El Portal Web Oficial del GAMC, desarrollado en tres laboratorios (HTML5 semántico y accesibilidad; JavaScript modular; organizador multitrámite, gráfico dinámico, modo oscuro y mapa de la Alcaldía), atacó la primera parte del problema: la información ya se puede consultar desde cualquier lugar con conexión.

Queda pendiente la segunda parte: que la relación entre el vecino y el municipio deje un registro. Hoy el portal informa, pero no recibe ni conserva solicitudes, y administrar su contenido requiere editar código. Esta propuesta plantea completar el sistema con un servidor y una base de datos reales, de modo que el portal pase de ser una vitrina informativa a una plataforma de atención ciudadana.

El documento se organiza así: tipo de proyecto, punto de partida y cliente; formulación del problema con su árbol de problemas; objetivos y justificación; factibilidad técnica, operativa y económica; alcances y límites; y el cronograma de ejecución.

---

## Tipo de proyecto

**Proyecto integrador.** Se resuelve un problema concreto de una institución real construyendo un producto de software funcional, acompañado de la documentación de ingeniería que respalda las decisiones tomadas.

No es una monografía: no se estudia el estado del arte de los portales de gobierno electrónico, se construye la parte que le falta a uno. No es una investigación: no se responde una pregunta abierta, se resuelve un problema conocido.

---

## El punto de partida

Este proyecto continúa el trabajo del módulo de Programación Web (Laboratorios 1, 2 y 3, versión 4.0 del portal), en el que se construyó el frontend completo.

| Ya existe | Se construye en este módulo |
|---|---|
| el frontend funcionando: HTML5 semántico, CSS3 con tema claro/oscuro y JavaScript modular (`datos.js`, `funciones.js`, `render.js`, `eventos.js`, `app.js`, `tablero.js`, `preferencias.js`) | la propuesta que lo justifica |
| el catálogo de 6 trámites escrito en el código (`datos.js` y `TRAMITES_CONFIG` en `tablero.js`) | una base de datos relacional MySQL y un servicio PHP que lo entrega |
| tickets generados en el navegador mediante *closures* (`crearGeneradorTickets()`) | registro de solicitudes en servidor con ticket correlativo único y persistente |
| un formulario de atención con validación por expresiones regulares, sin servidor que lo reciba | validación también en el servidor y almacenamiento de cada solicitud |
| "funciona en mi máquina" (ejecución en local) | requerimientos, pruebas, costos y *deploy* (puesta en producción) |

---

## El cliente

**Gobierno Autónomo Municipal de Colquencha (GAMC)**, Primera Sección de la Provincia Aroma, Departamento de La Paz. Su sede administrativa se encuentra en el centro cívico de la Plaza 28 de Agosto. Atiende trámites como catastro y certificación topográfica, licencias de funcionamiento, impuestos municipales, personería jurídica comunal y de ayllu, caminos vecinales y asistencia social a través del SLIM y la Defensoría de la Niñez.

**Cómo trabaja hoy:** el portal web publica el catálogo de trámites con sus requisitos, aranceles, oficinas responsables y tiempos estimados, un organizador para preparar la carpeta de documentos, los horarios de atención y la ubicación de la Alcaldía. El portal no registra solicitudes: la gestión efectiva de cada trámite continúa dependiendo de la atención presencial.

**Fuente de esta información:** Memoria Técnica y Descriptiva del proyecto (versión 4.0, octubre de 2026), que incluye el relevamiento de trámites, requisitos y aranceles y la solicitud institucional de depurar el portal para dejarlo en un formato limpio y directo. Responsable municipal que valida los requerimientos: [completar nombre y cargo].

---

## Formulación del problema

El Portal Web del GAMC resolvió el acceso a la información de trámites, pero su arquitectura actual es íntegramente de cliente: todo el contenido y toda la lógica residen en archivos HTML, CSS y JavaScript. Esto genera tres dificultades que se desprenden de la arquitectura documentada en la memoria técnica:

**Contenido atado al código.** Los trámites, requisitos, aranceles y oficinas responsables están definidos dentro de archivos JavaScript, y el catálogo aparece tanto en `datos.js` como en `TRAMITES_CONFIG` de `tablero.js`. Actualizar un arancel o agregar un requisito exige que un programador edite el código y vuelva a publicar el sitio, y existe el riesgo de que las dos copias queden desincronizadas.

**Ausencia de registro de solicitudes.** El formulario de atención ciudadana valida los datos con expresiones regulares, pero no existe un servidor que los reciba ni una base de datos donde guardarlos. El municipio no tiene forma de saber cuántas consultas llegan por el portal, de qué trámites ni de quiénes.

**Tickets sin respaldo ni unicidad.** Los identificadores de ticket se generan en el navegador de cada visitante con un generador correlativo en memoria. Dos ciudadanos distintos pueden recibir el mismo número, y al cerrar la pestaña el correlativo se pierde. El ticket no sirve como referencia para dar seguimiento a una solicitud.

### Árbol de problemas

```
   EFECTOS      El municipio no puede dar seguimiento a las solicitudes que llegan por el portal
                Información de trámites que puede quedar desactualizada o inconsistente
                El vecino que ya hizo la consulta debe igualmente acudir a ciegas a la Alcaldía
                          ↑
   PROBLEMA     El portal informa, pero no registra ni conserva solicitudes ni permite
   CENTRAL      administrar su contenido sin modificar código
                          ↑
   CAUSAS       El catálogo de trámites está escrito en archivos JavaScript
                No existe servidor ni base de datos
                Los tickets se generan en el navegador, sin correlativo único
```

### Problema central

> El Portal Web del GAMC no cuenta con un servidor ni una base de datos que
> permitan registrar y dar seguimiento a las solicitudes ciudadanas ni
> actualizar el catálogo de trámites sin modificar el código, lo que limita al
> portal a una función informativa.

---

## Objetivos

### Objetivo general

Desarrollar la plataforma de atención ciudadana del Portal Web del GAMC, que permita administrar el catálogo de trámites desde una base de datos y registrar y consultar solicitudes ciudadanas con ticket único.

### Objetivos específicos

1. Diseñar la base de datos relacional que registre trámites, requisitos, oficinas responsables y solicitudes de atención ciudadana.
2. Migrar el catálogo de 6 trámites desde los archivos JavaScript a la base de datos y entregarlo al frontend mediante un servicio PHP en formato JSON.
3. Implementar el registro de una solicitud de atención ciudadana desde el navegador, con validación en el servidor y asignación de un ticket correlativo único.
4. Implementar la consulta del estado de una solicitud por número de ticket y un panel de administración básico, con acceso por usuario y contraseña, para gestionar trámites y solicitudes.
5. Verificar el funcionamiento del sistema mediante una matriz de casos de prueba que cubra el registro, la consulta, la administración y los casos de error, y ponerlo en producción en un hosting.

---

## Justificación

**Práctica.** Las tres dificultades de la formulación tienen solución directa con una base de datos y un servidor. Un catálogo en base de datos se actualiza desde un panel sin tocar código; una solicitud almacenada puede consultarse y atenderse; y un ticket generado en el servidor es único y permite el seguimiento. Para el vecino, el portal deja de ser solo un lugar donde leer requisitos y pasa a ser un punto de contacto con trazabilidad.

**Técnica.** Se conserva la arquitectura web ya construida y se agrega el servidor, en lugar de rehacer el portal. Se usa PHP y MySQL porque funcionan sobre hosting compartido de bajo costo, el entorno de desarrollo (XAMPP) funciona sin conexión a internet y el frontend puede consumir el servicio con `fetch` sin cambios mayores. La memoria técnica destaca que el portal es ligero, sin librerías externas innecesarias; el backend mantiene ese criterio, importante para comunidades con conectividad limitada.

**Social.** El municipio atiende a comunidades y ayllus con barreras de conectividad geográfica. Un portal que registra la solicitud y permite verificar su estado reduce los desplazamientos innecesarios, porque el vecino puede saber en qué situación está su trámite antes de decidir si viaja.

---

## Factibilidad

**Técnica.** El sistema requiere un servidor con PHP 8 y MySQL, configuración estándar de cualquier hosting compartido. El desarrollo se hace con XAMPP en local. El frontend ya está construido y el equipo domina HTML5, CSS3 y JavaScript modular; la capa nueva (PHP, SQL y consumo de un servicio JSON) es acotada y es la parte de aprendizaje central del módulo.

**Operativa.** El sistema lo usarán dos perfiles: el ciudadano, que registra una solicitud y consulta su ticket desde el celular o una computadora, y el funcionario municipal, que revisa solicitudes y actualiza el catálogo desde el panel. La interfaz ciudadana ya existe y es accesible (WAI-ARIA, modo claro/oscuro, adaptación móvil); el panel se limita a dos acciones principales: listar solicitudes y cambiar su estado, y editar trámites.

*Riesgos identificados:*

- **Baja adopción por parte del personal.** Si el funcionario no revisa el panel, las solicitudes se acumulan sin respuesta. Se mitiga con un listado ordenado por fecha y estado pendiente, visible desde la primera pantalla del panel.
- **Conectividad limitada del ciudadano.** Se mitiga manteniendo el portal ligero y dejando que las páginas informativas sigan funcionando aunque el registro de solicitudes falle, con mensajes claros de error.
- **Datos personales.** El formulario recoge datos del ciudadano. Se recogen solo los necesarios, y la consulta por ticket no muestra datos personales completos.

**Económica.** El desarrollo se realiza como parte del módulo, sin costo para el municipio. La operación anual estimada es referencial y debe confirmarse con cotización o con el hosting que ya use la institución:

| Concepto | Costo anual (Bs) |
|---|---|
| Hosting compartido con PHP y MySQL | 350 (estimado) |
| Dominio | 0 a 250 (por confirmar) |
| **Total estimado** | **350 a 600** |

El rango del dominio depende de si el municipio ya cuenta con uno institucional o con infraestructura propia, en cuyo caso el costo adicional sería nulo.

---

## Alcances y límites

### Alcance

El sistema entregado incluye:

- base de datos relacional con trámites, requisitos, oficinas responsables y solicitudes
- servicio en PHP que entrega el catálogo de trámites al frontend desde la base de datos
- registro de una solicitud de atención ciudadana con validación en el servidor y ticket correlativo único
- consulta del estado de una solicitud por número de ticket
- panel de administración con acceso por usuario y contraseña para gestionar solicitudes y editar trámites, requisitos y aranceles
- respaldo del registro mediante exportación de la base de datos

### Límites

El sistema **no** incluye, y en cada caso se indica por qué:

- **Pago en línea de aranceles.** Los aranceles se cancelan en ventanilla (por ejemplo, en la Ventanilla Única de Recaudaciones), y una pasarela de pagos requiere convenios con entidades financieras, fuera del alcance de un módulo de cuatro semanas.
- **Tramitación completa en línea y firma digital.** El sistema registra la solicitud y su estado, pero la resolución del trámite sigue siendo presencial. Digitalizar el procedimiento administrativo implicaría cambios normativos y organizativos del municipio.
- **Integración con sistemas de otras entidades del Estado.** Requiere convenios y credenciales institucionales que escapan al proyecto.
- **Notificaciones automáticas por SMS o mensajería.** Requiere un servicio de mensajería de pago. Queda planteado como recomendación.
- **Aplicación móvil nativa.** El portal web adaptable cubre el uso desde teléfonos sin instalar nada.

---

## Cronograma

El módulo se desarrolla en cuatro semanas, de lunes a viernes, del **23 de noviembre al 18 de diciembre de 2026**. Cada semana cierra con un entregable verificable.

| Semana | Fechas | Actividades | Entregable | Objetivo |
|---|---|---|---|---|
| 1 | 23 – 27 nov | Levantamiento de requerimientos funcionales y no funcionales; modelo entidad-relación; script SQL de creación de la base de datos; instalación y configuración del entorno XAMPP | Documento de requerimientos, diagrama E-R y script SQL | 1 |
| 2 | 30 nov – 4 dic | Migración de los 6 trámites a MySQL; servicio PHP que entrega el catálogo en JSON; adaptación de `datos.js` y `tablero.js` para consumir el servicio | Catálogo servido desde la base de datos y consumido por el frontend | 2 |
| 3 | 7 – 11 dic | Registro de solicitudes con validación en servidor y ticket correlativo único; consulta de estado por ticket; panel de administración con acceso protegido | Flujo completo: solicitud → ticket → consulta → gestión desde el panel | 3 y 4 |
| 4 | 14 – 18 dic | Matriz de casos de prueba y correcciones; despliegue en hosting; exportación de respaldo; manual breve de uso; documento final y preparación de la defensa | Sistema en producción, matriz de pruebas ejecutada y documentación final | 5 |

### Hitos de control

| Fecha | Hito |
|---|---|
| 27 nov | Base de datos diseñada y creada |
| 4 dic | Catálogo servido desde la base de datos |
| 11 dic | Flujo completo de solicitud funcionando en local |
| 18 dic | Sistema en producción y documentación entregada |

### Plan de contingencia

Si alguna semana se retrasa, se priorizan los objetivos 1, 2 y 3, que sostienen el valor central del proyecto. El panel de administración (objetivo 4) se reduciría a su versión mínima: listar solicitudes, cambiar su estado y editar aranceles. La fecha de la defensa está pendiente de confirmación con la coordinación del programa.
