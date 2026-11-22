# PROPUESTA DE PROYECTO INTERMODULAR

EduHub

2º DAW Semipresencial, IES San Vicente, curso 2026-2027

Integrantes: Gloria Michelle Rosenberg, Sergio Fernández Ros, José Ángel García Martínez, Ricardo Andretta Hurtado, Diana Boiko y Fernando Pacheco Olivé

## 1. Título del proyecto

EduHub – Plataforma educativa para la gestión de tareas, entregas y calificaciones.

## 2. Descripción general

Problema. En muchos centros, el trabajo diario del aula se reparte entre varias herramientas: una para publicar tareas, otra para recoger entregas, hojas de cálculo para las notas y el correo para avisar al alumnado.

Solución. EduHub es una aplicación web para un centro educativo que reúne en un solo lugar la organización de asignaturas y grupos y el ciclo completo de una tarea (publicación, entrega y calificación). La plataforma ofrece de base toda la operativa educativa necesaria e incorpora un módulo de contratación y pago (en modo de pruebas) con consulta de tipos de cambio para cubrir los requisitos de pasarela de pago y API externa. Como ampliación opcional, solo si queda tiempo al final, se añadiría un asistente de IA que genera borradores de actividades para el profesorado.

Objetivos.

- Ofrecer un flujo completo y estable de tareas y calificaciones entre profesorado y alumnado.

- Investigar el uso de la IA en la plataforma y en el desarrollo y, solo como ampliación opcional final, integrarla en la preparación de actividades.

- Aplicar un sistema seguro de usuarios y roles, con autenticación propia y externa.

- Construir una aplicación internacionalizada, responsive y desplegada de forma reproducible.

- Aplicar buenas prácticas de pruebas, documentación, control de versiones y sostenibilidad.

Perfiles de usuario. Cada perfil accede con su propio menú y opciones:

| Perfil                   | Qué ve y qué puede hacer                                                                                                                                         |
| ------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Administrador del centro | Cambia el rol de los usuarios, crea asignaturas y grupos, asigna profesorado, matricula al alumnado y gestiona la contratación y pagos del módulo de ampliación. |
| Profesor                 | Ve los grupos que tiene asignados, crea tareas, revisa las entregas y califica.                                                                                  |
| Alumno                   | Ve las tareas de los grupos en los que está matriculado, entrega sus trabajos y consulta sus notas.                                                              |

La cuenta de administrador se crea al instalar la aplicación. Las cuentas nuevas se crean con el rol de alumno, y el administrador asigna el rol de profesor.

## 3. Funcionalidades principales

La priorización tiene cuatro niveles:

- P1 – Mínimo comprometido: incluye todos los requisitos mínimos del proyecto y solo la funcionalidad imprescindible para que EduHub sea una plataforma educativa: organización académica básica, ciclo de una tarea y contratación/pago con conversión de divisas. La IA no forma parte del P1. Todo el P1 es obligatorio. El anexo relaciona cada requisito mínimo con el lugar donde se cubre.

- P2 – Ampliaciones previstas: todo lo que no es requisito mínimo, salvo la IA. Solo se empieza cuando el P1 esté terminado, probado y desplegado.

- P3 – Ampliaciones futuras: se pueden abordar si sobra tiempo una vez terminado el P2.

- Opcional final – Inteligencia artificial: toda la implementación de IA en la aplicación. Es lo último que se aborda: después del P2 y de las ampliaciones P3 que se decida hacer. El proyecto se considera completo sin ella.

### Módulo 1: Usuarios, acceso y seguridad (P1)

- Registro con correo y contraseña, protegido con CAPTCHA (Cloudflare Turnstile) verificado en el backend.

- Verificación de la cuenta mediante correo con enlace de un solo uso. No se puede iniciar sesión sin verificar.

- Inicio de sesión con correo y contraseña, e inicio de sesión con Google (OAuth 2.0 / OpenID Connect).

- Sesión y seguridad JWT: autenticación mediante JSON Web Tokens firmados con tiempo de expiración definido. El token de acceso se transmite en la cabecera `Authorization: Bearer <token>` mediante un interceptor HTTP de Angular y se almacena en memoria/estado de sesión del cliente (sin depender de almacenamiento persistente inseguro); el cierre de sesión limpia el estado de autenticación en el cliente y redirige al acceso, mientras que las decisiones de expiración y revocación se concretan en el diseño técnico.

- Contraseñas almacenadas de forma segura con el algoritmo Argon2id.

- Control de acceso por estado (autenticado/no autenticado) y por rol: guards de rutas en Angular y decorators/guards de endpoints en NestJS.

- Perfil de usuario con idioma, zona horaria y moneda preferida.

### Módulo 2: Organización académica (P1)

- Asignaturas y grupos: el administrador da de alta, edita y elimina asignaturas y grupos. Cada grupo pertenece a una asignatura.

- Asignación de profesorado: el administrador asigna uno o varios profesores a cada grupo y puede quitarlos. Un profesor solo ve y gestiona los grupos que tiene asignados.

- Matrícula del alumnado: el administrador matricula al alumnado en los grupos de forma individual o mediante selección de una lista, y puede darlos de baja del grupo. Un alumno solo ve las tareas de los grupos en los que está matriculado.

- Roles: el administrador modifica el rol de un usuario entre alumno y profesor, ajustando sus permisos operacionales sin eliminar su identidad del sistema.

### Módulo 3: Tareas, entregas y calificaciones (P1)

- Tareas: el profesor crea tareas para uno de sus grupos, con título, enunciado y fecha límite.

- Fechas: se guardan en UTC en base de datos y se muestran en la zona horaria del usuario en la interfaz cliente.

- Entregas: el alumno sube su entrega como un archivo (PDF, imagen o ZIP, hasta 10 MB). Mientras no venza la fecha límite, se permite la entrega o sustitución del archivo.

- Calificación: el profesor ve, para cada tarea, la lista del grupo con quién ha entregado y quién no, descarga los archivos y asigna una nota numérica de 0 a 10.

- Consulta: el alumno ve sus tareas pendientes, entregadas y sus calificaciones.

### Módulo 4: Contratación, divisas y pagos (P1)

Este módulo cubre de forma sencilla y directa los requisitos obligatorios de pasarela de pago y consumo de API externa de divisas sin añadir lógica compleja de cuotas, porcentajes o caducidades en P1.

- Consulta de importe y divisas: muestra el importe base del servicio/ampliación en euros con el formato del idioma activo y su equivalencia en la moneda preferida del usuario (USD o GBP), obtenida de la API externa Frankfurter (tipos de cambio del BCE) con caché diaria en backend.

- Contratación: el administrador inicia el pago mediante Stripe Checkout en modo de pruebas.

- Confirmación por webhook: un webhook de Stripe confirma el pago, registra la transacción con su estado (pendiente, pagada o fallida) y actualiza el estado de contratación en el sistema.

- Notificación: envío de correo automático con el resumen del pago procesado.

### Requisitos transversales incluidos en el mínimo (P1)

| Requisito            | Cómo se cumple                                                                                                                                                                           |
| -------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Internacionalización | Interfaz en español e inglés, con cambio de idioma. Fechas, horas, números y monedas se formatean según el idioma (es-ES / en-US) y las fechas se ajustan a la zona horaria del usuario. |
| Correos automáticos  | Verificación de cuenta y confirmación de pago.                                                                                                                                           |
| API externa          | Frankfurter (tipos de cambio), consumida desde el backend. Es distinta de los servicios de Google, Stripe y CAPTCHA.                                                                     |
| API REST             | Backend NestJS que expone una API REST consumida por el frontend.                                                                                                                        |
| Diseño responsive    | Interfaz adaptada a móvil, tableta y escritorio con Tailwind CSS.                                                                                                                        |

### Ampliaciones previstas (P2)

- **Usuarios y Seguridad:** Recuperación de contraseña por correo, refresh tokens con rotación, invalidez explícita de JWT en servidor (lista negra/redis), búsqueda y filtrado avanzado de usuarios, activación/desactivación manual de cuentas.

- **Organización:** Cursos escolares, niveles educativos y periodos lectivos, configuración global del centro (país, zona horaria y moneda por defecto), matriculación masiva mediante selección múltiple con buscador en tiempo real, conservación de historial académico completo al cambiar de rol, paneles de inicio con resumen por rol (dashboards).

- **Tareas y entregas:** Entregas fuera de plazo autorizadas por el profesor (excepciones individuales), archivos adjuntos en el enunciado de la tarea, estados detallados de entrega, comentarios de retroalimentación en texto por parte del profesor, historial de rectificación de calificaciones, media por asignatura, calendario de entregas con festivos (API Nager.Date) y aviso si la fecha límite cae en festivo.

- **Evaluación:** Exámenes online con corrección automática y horarios de los grupos.

- **Pagos, planes y capacidad:** Modelos de planes escalonados, cuotas o límites de almacenamiento con medición porcentual y avisos automáticos (ej. al 80 %), bloqueo por cuota, lógica de caducidad y renovación por curso académico e historial detallado de transacciones para el administrador.

- **Correos y avisos:** Correos automáticos al publicar una tarea o una calificación, recordatorio 24 horas antes de que venza una entrega, notificaciones dentro de la aplicación y correos en el idioma del destinatario.

- **Calidad:** Pruebas E2E con Playwright, documentación de la API con OpenAPI (Swagger), accesibilidad y medición de la huella web con Lighthouse.

### Ampliaciones futuras (P3)

Varios centros con separación de datos, autoestudio para usuarios independientes, biblioteca digital y contenidos multimedia, acceso de familias, comunidad y votaciones, suscripciones recurrentes, gestión económica y gestión de personal.

### Ampliaciones opcional final: inteligencia artificial

Es lo último del proyecto: después del P2 y de las ampliaciones P3 que se decida hacer. Si no da tiempo, no se implementa y el proyecto sigue completo.

- Generador de actividades: el profesor genera un borrador de actividad (enunciado, preguntas y solución) a partir de un tema, un nivel y un idioma. Lo revisa, lo edita y lo publica como tarea del módulo 3.

- Asistente de dudas: asistente para el alumnado por asignatura, con historial y límite mensual de consultas.

- Seguridad: las llamadas al proveedor de IA se hacen solo desde el backend: la clave nunca llega al navegador y no se envían datos personales del alumnado.

- Ampliación de pago/P2: si se implementa, puede asociarse a funcionalidades o niveles avanzados definidos en fases posteriores.

## 4. Tecnologías a utilizar

| Área               | Tecnologías                                                                                                                                                                                                                                                                                                                                                            |
| ------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Frontend           | Angular, TypeScript, Tailwind CSS, Transloco (internacionalización), pipes de Angular para fechas y monedas.                                                                                                                                                                                                                                                           |
| Backend            | Node.js, NestJS, TypeScript, Passport (JWT y Google OAuth), argon2, class-validator.                                                                                                                                                                                                                                                                                   |
| Base de datos      | PostgreSQL con Prisma ORM (esquema y migraciones).                                                                                                                                                                                                                                                                                                                     |
| Servicios externos | IA (opcional): proveedor por API (OpenAI, Google Gemini o Anthropic Claude), elegido tras la investigación. Google Identity (OAuth 2.0 / OpenID Connect), Cloudflare Turnstile (CAPTCHA). Stripe Checkout en modo de pruebas, Frankfurter (tipos de cambio). Correo: Nodemailer con un proveedor SMTP. En desarrollo, Mailpit para capturar los correos sin enviarlos. |
| Infraestructura    | Docker, Docker Compose, Nginx, Apache con Passenger, Let's Encrypt (HTTPS). AWS: EC2 (Debian 12), S3, CloudFront, Elastic Beanstalk, RDS PostgreSQL, Lambda, API Gateway y Aurora Serverless v2.                                                                                                                                                                       |
| Calidad y equipo   | Git y GitHub, GitHub Actions (CI/CD), Jest (pruebas unitarias y de integración).                                                                                                                                                                                                                                                                                       |

## 5. Alcance y limitaciones

### 5.1. Qué incluye el proyecto

Aplicación: los 4 módulos P1 y los requisitos transversales del apartado 3, funcionando de principio a fin.

Despliegues (P1). La aplicación se prepara para varios entornos con la misma base de código: configuración por variables de entorno, backend sin estado (JWT) para poder escalar y almacenamiento de archivos intercambiable entre disco local y AWS S3.

- VPS con Docker Compose y Nginx, en nube pública (IaaS): VPS Debian 12 sobre AWS EC2. Contenedores: Nginx (sirve el build de Angular, redirige las rutas internas a index.html y hace de proxy inverso de /api hacia NestJS en el puerto 3000), backend NestJS y PostgreSQL con volumen persistente. Red interna entre servicios, dependencias definidas, solo los puertos 80/443 expuestos y HTTPS, necesario para el login con Google y el webhook de Stripe. Es el despliegue principal.

- VPS con Apache y Passenger: Apache sirve Angular desde el DocumentRoot, publica la API Node.js en /api mediante Passenger y redirige las rutas del frontend a index.html para evitar errores 404.

- Híbrido: backend y base de datos en un servidor local con Docker Compose; frontend publicado en S3 y distribuido con CloudFront; archivos de las entregas almacenados en S3.

- PaaS con escalado dinámico y CI/CD: backend en contenedor en AWS Elastic Beanstalk con autoescalado (de 1 a 3 instancias según la carga), base de datos en Amazon RDS PostgreSQL y frontend en S3 con CloudFront. GitHub Actions ejecuta las pruebas y despliega automáticamente al integrar en la rama principal.

- Serverless: backend NestJS adaptado a AWS Lambda detrás de API Gateway, base de datos Aurora Serverless v2 (PostgreSQL), archivos en S3 y frontend en CloudFront, con alta disponibilidad y escalado automático. Incluye una estimación de costes ante un crecimiento progresivo del servicio (por ejemplo, 1, 10 y 50 centros).

Relación entre los despliegues. Los cinco despliegues no son proyectos distintos: usan la misma aplicación y el mismo repositorio, y se realizan de manera progresiva a lo largo del calendario, de forma que cada uno reutiliza lo construido en el anterior y añade una sola novedad. Entre ellos solo cambia dónde se ejecuta cada pieza; el código es el mismo y se configura con variables de entorno.

| Despliegue (en orden)                    | Frontend              | Backend                                          | Base de datos                    | Archivos        | Qué reutiliza y qué aporta                                                                                                                     |
| ---------------------------------------- | --------------------- | ------------------------------------------------ | -------------------------------- | --------------- | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| 1\. VPS con Docker Compose y Nginx (EC2) | Nginx en contenedor   | NestJS en contenedor                             | PostgreSQL en contenedor         | Disco (volumen) | Es la base: crea las imágenes Docker, el docker-compose y el manual. Aporta la nube pública con máquinas virtuales (IaaS).                     |
| 2\. VPS con Apache y Passenger           | Apache (DocumentRoot) | Node.js con Passenger                            | PostgreSQL en el VPS             | Disco local     | Reutiliza los mismos builds de Angular y NestJS, sin contenedores. Aporta la alternativa con servidor web clásico.                             |
| 3\. Híbrido                              | S3 + CloudFront       | NestJS en contenedor (servidor local)            | PostgreSQL en contenedor (local) | S3              | Reutiliza el docker-compose del 1. Aporta la combinación local + nube y el almacenamiento en S3.                                               |
| 4\. PaaS con CI/CD                       | S3 + CloudFront       | Contenedor en Elastic Beanstalk (1-3 instancias) | RDS PostgreSQL                   | S3              | Reutiliza la imagen del backend y el frontend en S3 del 3. Aporta autoescalado y despliegue automático con GitHub Actions.                     |
| 5\. Serverless                           | S3 + CloudFront       | NestJS en Lambda + API Gateway                   | Aurora Serverless v2             | S3              | Reutiliza el frontend y S3 del 4 y el mismo código del backend con un adaptador para Lambda. Aporta el pago por uso y la estimación de costes. |

Valoración de infraestructuras: con los datos reales de los cinco despliegues (coste, rendimiento y esfuerzo de puesta en marcha), se compara la nube pública con el despliegue local y un VPS en OVH según requisitos legales (RGPD, datos de menores, ubicación de los datos en la UE), seguridad y confidencialidad, coste de computación y transferencia, requisitos tecnológicos e integración o convivencia con otras infraestructuras.

Documentación: análisis y diseño (requisitos, casos de uso, modelo de datos, arquitectura y API), plan y resultados de pruebas, manual de usuario, y manual técnico de despliegue que permita reproducir el entorno desde cero en otro VPS (requisitos previos, instalación, configuración y puesta en marcha), con una guía de cada despliegue cloud.

Pruebas: pruebas unitarias de la lógica crítica (autenticación, permisos, zonas horarias y monedas) y pruebas de integración de la API con una base de datos de pruebas, ejecutadas en GitHub Actions.

Investigación de IA: comparación de proveedores para la plataforma (calidad en español, coste, límites y privacidad) y de asistentes de IA para el desarrollo (programación, pruebas y documentación), con una justificación de las herramientas elegidas. Es una investigación documental: no obliga a implementar la IA en la aplicación.

Sostenibilidad (ODS 9, 12 y 13):

- Carga diferida de rutas en Angular, imágenes optimizadas y compresión en Nginx, para transferir menos datos (ODS 12).

- Caché diaria de los tipos de cambio y consultas con índices, para evitar peticiones y cómputo innecesarios (ODS 12 y 13).

- Si se implementa la IA opcional, uso solo bajo demanda y con respuestas de longitud limitada, para reducir el consumo del servicio con más impacto energético (ODS 13).

- Imágenes de contenedor ligeras e instancias dimensionadas a la carga real (ODS 9 y 13).

### 5.2. Qué no incluye

- Cobros reales: la pasarela funciona solo en modo de pruebas.

- Entrenamiento de modelos de IA propios: se consumen servicios externos por API.

- Aplicación móvil nativa: la web responsive cubre el uso en móvil.

- Kubernetes: como servicio PaaS se usa AWS Elastic Beanstalk.

### 5.3. Limitaciones y criterio de cierre

- Los servicios externos se usan en sus planes gratuitos o de prueba, con sus límites.

- Los entornos cloud se crean con los créditos de AWS disponibles. Tras validarlos y documentarlos, pueden apagarse para no generar costes y volver a levantarse siguiendo el manual.

- Se trabaja únicamente con datos ficticios, nunca con datos reales de alumnado menor de edad.

Criterio de cierre: el proyecto se considera completo cuando todo el P1, incluidos los cinco despliegues, funciona y está probado y documentado. Ninguna funcionalidad P1 puede sustituirse por una P2. La IA es opcional y no forma parte del criterio de cierre.

## 6. Cronograma preliminar

De finales de noviembre de 2026 a junio de 2027. Las pruebas y la documentación se hacen de forma continua en cada fase. El primer despliegue en el VPS se realiza de forma temprana en enero para evitar cuellos de botella finales de infraestructura. Con seis personas, la responsabilidad se distribuye por módulos y capas técnicas (frontend, backend, BD y despliegues cloud) con puntos de integración periódicos para evitar dependencias de un único integrante. Las funcionalidades P1 deben estar terminadas a finales de abril, reservando mayo para validar despliegues cloud avanzados, consolidar pruebas y documentación, y junio para la entrega final.

| Fase                               | Nov | Dic | Ene | Feb | Mar | Abr | May | Jun | Resultado                                                               |
| ---------------------------------- | --- | --- | --- | --- | --- | --- | --- | --- | ----------------------------------------------------------------------- |
| 1\. Análisis y diseño              | ■   | ■   |     |     |     |     |     |     | Requisitos, casos de uso, modelo de datos, prototipos y API             |
| 2\. Base técnica                   |     | ■   |     |     |     |     |     |     | Repositorio, Docker Compose, CI e i18n                                  |
| 3\. M1 Usuarios y seguridad        |     |     | ■   |     |     |     |     |     | Hito 1: acceso completo desplegado en el VPS                            |
| 4\. M2 Organización académica      |     |     |     | ■   |     |     |     |     | Asignaturas, grupos y matrícula                                         |
| 5\. M3 Tareas y calificaciones     |     |     |     | □   | ■   |     |     |     | Hito 2: ciclo completo de una tarea                                     |
| 6\. M4 Contratación y pagos        |     |     |     |     | □   | ■   |     |     | Hito 3 (finales de abril): funcionalidades P1 completas                 |
| 7\. Despliegues en VPS             |     |     | ■   | ■   | ■   |     |     |     | Docker Compose con Nginx (enero) y Apache con Passenger (febrero-marzo) |
| 8\. Despliegues cloud y valoración |     |     |     |     | □   | ■   | ■   |     | Híbrido, PaaS con CI/CD, serverless, estimación de costes y comparación |
| 9\. Pruebas, P2 y documentación    |     |     | □   | □   | □   | □   | ■   |     | Hito 4 (finales de mayo): todo el P1 desplegado, probado y documentado  |
| 10\. IA (opcional)                 |     |     |     |     |     |     |     | □   | Solo al final, si queda tiempo                                          |
| 11\. Entrega y presentación        |     |     |     |     |     |     |     | ■   | Corrección final y defensa                                              |

Color oscuro: dedicación principal. Color claro: inicio o trabajo parcial. Las vacaciones de Navidad y Semana Santa sirven de margen para recuperar retrasos.

## Anexo. Requisitos mínimos del proyecto (todos con prioridad P1)

| Requisito mínimo                                                | Dónde se cubre                                                              | Prioridad |
| --------------------------------------------------------------- | --------------------------------------------------------------------------- | --------- |
| Login                                                           | M1                                                                          | P1        |
| Perfiles de usuario con opciones diferentes                     | M1 y M2: administrador, profesor y alumno, cada uno con su menú y opciones  | P1        |
| Conexión con base de datos                                      | PostgreSQL con Prisma ORM                                                   | P1        |
| Front en Angular                                                | Angular con TypeScript                                                      | P1        |
| Back con un framework                                           | NestJS                                                                      | P1        |
| CAPTCHA en el registro                                          | M1: Cloudflare Turnstile verificado en el backend                           | P1        |
| Login con sistemas externos                                     | M1: Google (OAuth 2.0 / OpenID Connect)                                     | P1        |
| Pasarela de pago                                                | M4: Stripe Checkout en modo de pruebas, con webhook                         | P1        |
| Emails automatizados                                            | M1 y M4: verificación de cuenta y confirmación de pago                      | P1        |
| API REST consumida por el front                                 | Backend NestJS                                                              | P1        |
| Internacionalización (2 idiomas, fechas, monedas, zona horaria) | Español e inglés; zona horaria por usuario (M3); monedas (M4)               | P1        |
| Contraseñas cifradas con un sistema seguro                      | M1: Argon2id                                                                | P1        |
| Verificación del registro por correo                            | M1: enlace de un solo uso                                                   | P1        |
| API externa para enriquecer los datos                           | M4: Frankfurter (tipos de cambio)                                           | P1        |
| Documentación de análisis y diseño                              | Apartado 5.1: requisitos, casos de uso, modelo de datos, arquitectura y API | P1        |
| Realización de pruebas                                          | Apartado 5.1: unitarias y de integración                                    | P1        |
| Manual de usuario                                               | Apartado 5.1                                                                | P1        |
| Despliegue en un servidor                                       | Apartado 5.1: VPS con Docker Compose y Nginx                                | P1        |
| Investigación de IAs                                            | Apartado 5.1: proveedores para la plataforma y para el desarrollo           | P1        |
| Sostenibilidad (ODS 9, 12 y 13)                                 | Apartado 5.1: medidas concretas                                             | P1        |
| Registro y login con token en el cliente                        | M1: JWT con interceptor de Angular                                          | P1        |
| Control de acceso a rutas y funciones                           | M1: guards por estado y rol en Angular y NestJS                             | P1        |
| Manual técnico de despliegue reproducible en otro VPS           | Apartado 5.1: manual desde cero                                             | P1        |
| Valoración nube frente a local u OVH                            | Apartado 5.1: legal, seguridad, confidencialidad, coste y tecnología        | P1        |
| Despliegue híbrido (local + S3, CloudFront)                     | Apartado 5.1: despliegue híbrido                                            | P1        |
| Despliegue en nube pública con máquinas virtuales               | Apartado 5.1: VPS sobre AWS EC2                                             | P1        |
| Despliegue PaaS con escalado dinámico y CI/CD                   | Apartado 5.1: Elastic Beanstalk y GitHub Actions                            | P1        |
| Despliegue serverless con estimación de costes                  | Apartado 5.1: Lambda, API Gateway y Aurora Serverless v2                    | P1        |
| Despliegue en VPS con Docker Compose                            | Apartado 5.1: frontend, backend y BD en contenedores                        | P1        |
| Apache con Passenger, /api e index.html                         | Apartado 5.1: despliegue en VPS con Apache                                  | P1        |
| Nginx como proxy inverso al puerto 3000 y /api                  | Apartado 5.1: contenedor Nginx del despliegue principal                     | P1        |
| Diseño web responsive                                           | Interfaz adaptada a móvil, tableta y escritorio                             | P1        |
| Framework de diseño                                             | Tailwind CSS                                                                | P1        |
