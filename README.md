# Documentación del Flujo de Ingreso de Datos para el Esquema de Base de Datos

## Introducción
Este documento describe el flujo de ingreso de datos para un esquema de base de datos relacional diseñado en MySQL para un sistema que gestiona presupuestos, cursos de aprendizaje y comunidades. El esquema incluye tablas para usuarios, roles, categorías financieras, presupuestos, cursos, rutas de aprendizaje, lecciones, progreso de usuarios, comunidades, mensajes, etiquetas, favoritos y registros de auditoría. La documentación detalla cada tabla, sus campos, relaciones, procesos de ingreso de datos, consideraciones de integridad y reglas de negocio relevantes.

El objetivo es proporcionar una guía clara para desarrolladores, administradores de bases de datos y usuarios finales, asegurando que el ingreso de datos sea consistente, seguro y cumpla con las reglas de negocio. La fecha de referencia para este documento es 30 de mayo de 2025.

---

## Descripción de las Tablas

### 1. roles
**Descripción**: Almacena los roles de los usuarios en el sistema (e.g., Administrador, Miembro). Los roles definen permisos y funciones dentro de la aplicación.

**Campos**:
| Campo | Tipo de Dato | Propósito | Restricciones | Ejemplo |
|-------|--------------|-----------|--------------|---------|
| `role_id` | INT | Identificador único del rol | PK, AUTO_INCREMENT | 1 |
| `name` | VARCHAR(50) | Nombre del rol | NOT NULL, UNIQUE | "Administrador" |
| `description` | VARCHAR(255) | Descripción del rol | NULL | "Usuario con permisos completos" |
| `is_active` | BOOLEAN | Indica si el rol está activo | NOT NULL, DEFAULT TRUE | TRUE |
| `created_at` | TIMESTAMP | Fecha y hora de creación | DEFAULT CURRENT_TIMESTAMP | 2025-05-30 13:42:00 |
| `updated_at` | TIMESTAMP | Fecha y hora de última actualización | DEFAULT CURRENT_TIMESTAMP ON UPDATE | 2025-05-30 13:42:00 |

### 2. users
**Descripción**: Almacena la información de los usuarios del sistema, incluyendo credenciales, perfil y estado.

**Campos**:
| Campo | Tipo de Dato | Propósito | Restricciones | Ejemplo |
|-------|--------------|-----------|--------------|---------|
| `user_id` | INT | Identificador único del usuario | PK, AUTO_INCREMENT | 1 |
| `full_name` | VARCHAR(100) | Nombre completo del usuario | NOT NULL | "Juan Pérez" |
| `email` | VARCHAR(255) | Correo electrónico | NOT NULL, UNIQUE, CHECK (formato email) | "juan.perez@example.com" |
| `password` | VARCHAR(255) | Contraseña hasheada | NOT NULL | "$2b$10$..." (bcrypt hash) |
| `profile_picture` | VARCHAR(255) | URL de la foto de perfil | NOT NULL | "https://example.com/profiles/juan.jpg" |
| `role_id` | INT | ID del rol asignado | NOT NULL, FK (roles) | 1 |
| `registration_date` | TIMESTAMP | Fecha de registro | DEFAULT CURRENT_TIMESTAMP | 2025-05-30 13:42:00 |
| `last_accessed` | TIMESTAMP | Último acceso al sistema | NULL | 2025-05-30 13:42:00 |
| `is_active` | BOOLEAN | Indica si el usuario está activo | NOT NULL, DEFAULT TRUE | TRUE |
| `deleted_at` | TIMESTAMP | Fecha de eliminación lógica | NULL | NULL |
| `created_at` | TIMESTAMP | Fecha de creación | DEFAULT CURRENT_TIMESTAMP | 2025-05-30 13:42:00 |
| `created_by` | INT | ID del usuario que creó el registro | NOT NULL, FK (users) | 1 |
| `updated_at` | TIMESTAMP | Fecha de última actualización | DEFAULT CURRENT_TIMESTAMP ON UPDATE | 2025-05-30 13:42:00 |
| `updated_by` | INT | ID del usuario que actualizó | NOT NULL, FK (users) | 1 |

### 3. categories
**Descripción**: Almacena categorías financieras para clasificar transacciones y presupuestos (e.g., Ingresos, Gastos).

**Campos**:
| Campo | Tipo de Dato | Propósito | Restricciones | Ejemplo |
|-------|--------------|-----------|--------------|---------|
| `category_id` | INT | Identificador único | PK, AUTO_INCREMENT | 1 |
| `name` | VARCHAR(50) | Nombre de la categoría | NOT NULL, UNIQUE | "Alimentación" |
| `description` | TEXT | Descripción de la categoría | NULL | "Gastos en comida y bebidas" |
| `type` | ENUM('Income', 'Expense', 'Savings') | Tipo de categoría | NOT NULL | "Expense" |
| `priority` | INT | Prioridad para ordenamiento | DEFAULT 0 | 1 |
| `is_active` | BOOLEAN | Estado activo | NOT NULL, DEFAULT TRUE | TRUE |
| `deleted_at` | TIMESTAMP | Fecha de eliminación lógica | NULL | NULL |
| `created_at` | TIMESTAMP | Fecha de creación | DEFAULT CURRENT_TIMESTAMP | 2025-05-30 13:42:00 |
| `created_by` | INT | ID del usuario creador | NOT NULL, FK (users) | 1 |
| `updated_at` | TIMESTAMP | Fecha de última actualización | DEFAULT CURRENT_TIMESTAMP ON UPDATE | 2025-05-30 13:42:00 |
| `updated_by` | INT | ID del usuario que actualizó | NOT NULL, FK (users) | 1 |

### 4. subcategories
**Descripción**: Subdivide las categorías para una gestión financiera más granular (e.g., "Comida rápida" bajo "Alimentación").

**Campos**:
| Campo | Tipo de Dato | Propósito | Restricciones | Ejemplo |
|-------|--------------|-----------|--------------|---------|
| `subcategory_id` | INT | Identificador único | PK, AUTO_INCREMENT | 1 |
| `user_id` | INT | ID del usuario propietario | NOT NULL, FK (users) | 1 |
| `category_id` | INT | ID de la categoría padre | NOT NULL, FK (categories) | 1 |
| `name` | VARCHAR(50) | Nombre de la subcategoría | NOT NULL | "Comida rápida" |
| `is_active` | BOOLEAN | Estado activo | NOT NULL, DEFAULT TRUE | TRUE |
| `deleted_at` | TIMESTAMP | Fecha de eliminación lógica | NULL | NULL |
| `created_at` | TIMESTAMP | Fecha de creación | DEFAULT CURRENT_TIMESTAMP | 2025-05-30 13:42:00 |
| `created_by` | INT | ID del usuario creador | NOT NULL, FK (users) | 1 |
| `updated_at` | TIMESTAMP | Fecha de última actualización | DEFAULT CURRENT_TIMESTAMP ON UPDATE | 2025-05-30 13:42:00 |
| `updated_by` | INT | ID del usuario que actualizó | NOT NULL, FK (users) | 1 |

### 5. budgets
**Descripción**: Gestiona presupuestos mensuales por usuario, definiendo límites financieros.

**Campos**:
| Campo | Tipo de Dato | Propósito | Restricciones | Ejemplo |
|-------|--------------|-----------|--------------|---------|
| `budget_id` | INT | Identificador único | PK, AUTO_INCREMENT | 1 |
| `user_id` | INT | ID del usuario | NOT NULL, FK (users) | 1 |
| `month_year` | DATE | Mes y año del presupuesto | NOT NULL | 2025-05-01 |
| `status` | ENUM('Open', 'Closed') | Estado del presupuesto | NOT NULL, DEFAULT 'Open' | "Open" |
| `budget_limit` | DECIMAL(10,2) | Límite total del presupuesto | DEFAULT 0.00 | 1000.00 |
| `is_active` | BOOLEAN | Estado activo | NOT NULL, DEFAULT TRUE | TRUE |
| `deleted_at` | TIMESTAMP | Fecha de eliminación lógica | NULL | NULL |
| `created_at` | TIMESTAMP | Fecha de creación | DEFAULT CURRENT_TIMESTAMP | 2025-05-30 13:42:00 |
| `created_by` | INT | ID del usuario creador | NOT NULL, FK (users) | 1 |
| `updated_at` | TIMESTAMP | Fecha de última actualización | DEFAULT CURRENT_TIMESTAMP ON UPDATE | 2025-05-30 13:42:00 |
| `updated_by` | INT | ID del usuario que actualizó | NOT NULL, FK (users) | 1 |

### 6. budget_allocations
**Descripción**: Asigna montos presupuestados a subcategorías dentro de un presupuesto.

**Campos**:
| Campo | Tipo de Dato | Propósito | Restricciones | Ejemplo |
|-------|--------------|-----------|--------------|---------|
| `allocation_id` | INT | Identificador único | PK, AUTO_INCREMENT | 1 |
| `budget_id` | INT | ID del presupuesto | NOT NULL, FK (budgets) | 1 |
| `subcategory_id` | INT | ID de la subcategoría | NOT NULL, FK (subcategories) | 1 |
| `assigned_amount` | DECIMAL(10,2) | Monto asignado | NOT NULL, CHECK (>= 0) | 200.00 |
| `spent_amount` | DECIMAL(10,2) | Monto gastado | DEFAULT 0.00, CHECK (>= 0) | 150.00 |
| `is_active` | BOOLEAN | Estado activo | NOT NULL, DEFAULT TRUE | TRUE |
| `deleted_at` | TIMESTAMP | Fecha de eliminación lógica | NULL | NULL |
| `created_at` | TIMESTAMP | Fecha de creación | DEFAULT CURRENT_TIMESTAMP | 2025-05-30 13:42:00 |
| `created_by` | INT | ID del usuario creador | NOT NULL, FK (users) | 1 |
| `updated_at` | TIMESTAMP | Fecha de última actualización | DEFAULT CURRENT_TIMESTAMP ON UPDATE | 2025-05-30 13:42:00 |
| `updated_by` | INT | ID del usuario que actualizó | NOT NULL, FK (users) | 1 |

### 7. transactions
**Descripción**: Registra transacciones financieras asociadas a un usuario, presupuesto y subcategoría.

**Campos**:
| Campo | Tipo de Dato | Propósito | Restricciones | Ejemplo |
|-------|--------------|-----------|--------------|---------|
| `transaction_id` | INT | Identificador único | PK, AUTO_INCREMENT | 1 |
| `user_id` | INT | ID del usuario | NOT NULL, FK (users) | 1 |
| `budget_id` | INT | ID del presupuesto | NOT NULL, FK (budgets) | 1 |
| `subcategory_id` | INT | ID de la subcategoría | NOT NULL, FK (subcategories) | 1 |
| `transaction_date` | TIMESTAMP | Fecha de la transacción | NOT NULL | 2025-05-30 13:42:00 |
| `description` | TEXT | Descripción de la transacción | NULL | "Compra en supermercado" |
| `amount` | DECIMAL(10,2) | Monto de la transacción | NOT NULL, CHECK (>= 0) | 50.00 |
| `transaction_type` | ENUM('Credit', 'Debit') | Tipo de transacción | NOT NULL | "Debit" |
| `is_active` | BOOLEAN | Estado activo | NOT NULL, DEFAULT TRUE | TRUE |
| `deleted_at` | TIMESTAMP | Fecha de eliminación lógica | NULL | NULL |
| `created_at` | TIMESTAMP | Fecha de creación | DEFAULT CURRENT_TIMESTAMP | 2025-05-30 13:42:00 |
| `created_by` | INT | ID del usuario creador | NOT NULL, FK (users) | 1 |
| `updated_at` | TIMESTAMP | Fecha de última actualización | DEFAULT CURRENT_TIMESTAMP ON UPDATE | 2025-05-30 13:42:00 |
| `updated_by` | INT | ID del usuario que actualizó | NOT NULL, FK (users) | 1 |

### 8. routes
**Descripción**: Define rutas de aprendizaje, que agrupan cursos en un orden específico.

**Campos**:
| Campo | Tipo de Dato | Propósito | Restricciones | Ejemplo |
|-------|--------------|-----------|--------------|---------|
| `route_id` | INT | Identificador único | PK, AUTO_INCREMENT | 1 |
| `title` | VARCHAR(50) | Título de la ruta | NOT NULL, UNIQUE | "Ruta de Finanzas Personales" |
| `description` | TEXT | Descripción de la ruta | NULL | "Aprende a gestionar tus finanzas" |
| `is_active` | BOOLEAN | Estado activo | NOT NULL, DEFAULT TRUE | TRUE |
| `deleted_at` | TIMESTAMP | Fecha de eliminación lógica | NULL | NULL |
| `created_at` | TIMESTAMP | Fecha de creación | DEFAULT CURRENT_TIMESTAMP | 2025-05-30 13:42:00 |
| `created_by` | INT | ID del usuario creador | NOT NULL, FK (users) | 1 |
| `updated_at` | TIMESTAMP | Fecha de última actualización | DEFAULT CURRENT_TIMESTAMP ON UPDATE | 2025-05-30 13:42:00 |
| `updated_by` | INT | ID del usuario que actualizó | NOT NULL, FK (users) | 1 |

### 9. courses
**Descripción**: Almacena cursos de aprendizaje con sus metadatos.

**Campos**:
| Campo | Tipo de Dato | Propósito | Restricciones | Ejemplo |
|-------|--------------|-----------|--------------|---------|
| `course_id` | INT | Identificador único | PK, AUTO_INCREMENT | 1 |
| `title` | VARCHAR(50) | Título del curso | NOT NULL, UNIQUE | "Presupuesto 101" |
| `description` | TEXT | Descripción del curso | NULL | "Introducción a la planificación financiera" |
| `image_url` | VARCHAR(255) | URL de la imagen del curso | NOT NULL | "https://example.com/courses/presupuesto.jpg" |
| `is_active` | BOOLEAN | Estado activo | NOT NULL, DEFAULT TRUE | TRUE |
| `deleted_at` | TIMESTAMP | Fecha de eliminación lógica | NULL | NULL |
| `created_at` | TIMESTAMP | Fecha de creación | DEFAULT CURRENT_TIMESTAMP | 2025-05-30 13:42:00 |
| `created_by` | INT | ID del usuario creador | NOT NULL, FK (users) | 1 |
| `updated_at` | TIMESTAMP | Fecha de última actualización | DEFAULT CURRENT_TIMESTAMP ON UPDATE | 2025-05-30 13:42:00 |
| `updated_by` | INT | ID del usuario que actualizó | NOT NULL, FK (users) | 1 |

### 10. route_courses
**Descripción**: Relaciona rutas y cursos, definiendo el orden de los cursos en una ruta.

**Campos**:
| Campo | Tipo de Dato | Propósito | Restricciones | Ejemplo |
|-------|--------------|-----------|--------------|---------|
| `route_course_id` | INT | Identificador único | PK, AUTO_INCREMENT | 1 |
| `order_in_route` | INT | Orden del curso en la ruta | NOT NULL | 1 |
| `route_id` | INT | ID de la ruta | NOT NULL, FK (routes) | 1 |
| `course_id` | INT | ID del curso | NOT NULL, FK (courses) | 1 |
| `is_active` | BOOLEAN | Estado activo | NOT NULL, DEFAULT TRUE | TRUE |
| `deleted_at` | TIMESTAMP | Fecha de eliminación lógica | NULL | NULL |
| `created_at` | TIMESTAMP | Fecha de creación | DEFAULT CURRENT_TIMESTAMP | 2025-05-30 13:42:00 |
| `created_by` | INT | ID del usuario creador | NOT NULL, FK (users) | 1 |
| `updated_at` | TIMESTAMP | Fecha de última actualización | DEFAULT CURRENT_TIMESTAMP ON UPDATE | 2025-05-30 13:42:00 |
| `updated_by` | INT | ID del usuario que actualizó | NOT NULL, FK (users) | 1 |

### 11. lessons
**Descripción**: Almacena lecciones individuales dentro de un curso.

**Campos**:
| Campo | Tipo de Dato | Propósito | Restricciones | Ejemplo |
|-------|--------------|-----------|--------------|---------|
| `lesson_id` | INT | Identificador único | PK, AUTO_INCREMENT | 1 |
| `course_id` | INT | ID del curso | NOT NULL, FK (courses) | 1 |
| `title` | VARCHAR(50) | Título de la lección | NOT NULL | "Introducción al Presupuesto" |
| `video_url` | VARCHAR(255) | URL del video de la lección | NULL | "https://example.com/videos/lesson1.mp4" |
| `order_in_course` | INT | Orden en el curso | NOT NULL, UNIQUE (con course_id) | 1 |
| `lesson_type` | ENUM('Standard', 'Premium') | Tipo de lección | NOT NULL, DEFAULT 'Standard' | "Standard" |
| `duration` | INT | Duración en segundos | NULL | 300 |
| `version` | INT | Versión de la lección | NOT NULL, DEFAULT 1 | 1 |
| `is_active` | BOOLEAN | Estado activo | NOT NULL, DEFAULT TRUE | TRUE |
| `deleted_at` | TIMESTAMP | Fecha de eliminación lógica | NULL | NULL |
| `created_at` | TIMESTAMP | Fecha de creación | DEFAULT CURRENT_TIMESTAMP | 2025-05-30 13:42:00 |
| `created_by` | INT | ID del usuario creador | NOT NULL, FK (users) | 1 |
| `updated_at` | TIMESTAMP | Fecha de última actualización | DEFAULT CURRENT_TIMESTAMP ON UPDATE | 2025-05-30 13:42:00 |
| `updated_by` | INT | ID del usuario que actualizó | NOT NULL, FK (users) | 1 |

### 12. questions
**Descripción**: Almacena preguntas asociadas a lecciones para evaluaciones.

**Campos**:
| Campo | Tipo de Dato | Propósito | Restricciones | Ejemplo |
|-------|--------------|-----------|--------------|---------|
| `question_id` | INT | Identificador único | PK, AUTO_INCREMENT | 1 |
| `lesson_id` | INT | ID de la lección | NOT NULL, FK (lessons) | 1 |
| `text` | TEXT | Texto de la pregunta | NOT NULL | "¿Qué es un presupuesto?" |
| `question_type` | ENUM('Multiple Choice', 'True/False', 'Open Ended') | Tipo de pregunta | NOT NULL | "Multiple Choice" |
| `is_active` | BOOLEAN | Estado activo | NOT NULL, DEFAULT TRUE | TRUE |
| `deleted_at` | TIMESTAMP | Fecha de eliminación lógica | NULL | NULL |
| `created_at` | TIMESTAMP | Fecha de creación | DEFAULT CURRENT_TIMESTAMP | 2025-05-30 13:42:00 |
| `created_by` | INT | ID del usuario creador | NOT NULL, FK (users) | 1 |
| `updated_at` | TIMESTAMP | Fecha de última actualización | DEFAULT CURRENT_TIMESTAMP ON UPDATE | 2025-05-30 13:42:00 |
| `updated_by` | INT | ID del usuario que actualizó | NOT NULL, FK (users) | 1 |

### 13. answers
**Descripción**: Almacena respuestas posibles para las preguntas.

**Campos**:
| Campo | Tipo de Dato | Propósito | Restricciones | Ejemplo |
|-------|--------------|-----------|--------------|---------|
| `answer_id` | INT | Identificador único | PK, AUTO_INCREMENT | 1 |
| `question_id` | INT | ID de la pregunta | NOT NULL, FK (questions) | 1 |
| `text` | TEXT | Texto de la respuesta | NOT NULL | "Un plan financiero" |
| `is_correct` | BOOLEAN | Indica si es correcta | NOT NULL | TRUE |
| `is_active` | BOOLEAN | Estado activo | NOT NULL, DEFAULT TRUE | TRUE |
| `deleted_at` | TIMESTAMP | Fecha de eliminación lógica | NULL | NULL |
| `created_at` | TIMESTAMP | Fecha de creación | DEFAULT CURRENT_TIMESTAMP | 2025-05-30 13:42:00 |
| `created_by` | INT | ID del usuario creador | NOT NULL, FK (users) | 1 |
| `updated_at` | TIMESTAMP | Fecha de última actualización | DEFAULT CURRENT_TIMESTAMP ON UPDATE | 2025-05-30 13:42:00 |
| `updated_by` | INT | ID del usuario que actualizó | NOT NULL, FK (users) | 1 |

### 14. lesson_progress
**Descripción**: Registra el progreso de los usuarios en las lecciones.

**Campos**:
| Campo | Tipo de Dato | Propósito | Restricciones | Ejemplo |
|-------|--------------|-----------|--------------|---------|
| `progress_id` | INT | Identificador único | PK, AUTO_INCREMENT | 1 |
| `user_id` | INT | ID del usuario | NOT NULL, FK (users) | 1 |
| `lesson_id` | INT | ID de la lección | NOT NULL, FK (lessons) | 1 |
| `lesson_version` | INT | Versión de la lección | NOT NULL, DEFAULT 1 | 1 |
| `status` | ENUM('Not Started', 'In Progress', 'Completed') | Estado del progreso | NOT NULL, DEFAULT 'Not Started' | "In Progress" |
| `progress_date` | TIMESTAMP | Fecha de actualización del progreso | DEFAULT CURRENT_TIMESTAMP | 2025-05-30 13:42:00 |
| `completion_percentage` | DECIMAL(4,1) | Porcentaje de completitud | DEFAULT 0.0, CHECK (0-100) | 50.0 |
| `is_active` | BOOLEAN | Estado activo | NOT NULL, DEFAULT TRUE | TRUE |
| `deleted_at` | TIMESTAMP | Fecha de eliminación lógica | NULL | NULL |
| `created_at` | TIMESTAMP | Fecha de creación | DEFAULT CURRENT_TIMESTAMP | 2025-05-30 13:42:00 |
| `created_by` | INT | ID del usuario creador | NOT NULL, FK (users) | 1 |
| `updated_at` | TIMESTAMP | Fecha de última actualización | DEFAULT CURRENT_TIMESTAMP ON UPDATE | 2025-05-30 13:42:00 |
| `updated_by` | INT | ID del usuario que actualizó | NOT NULL, FK (users) | 1 |

### 15. communities
**Descripción**: Gestiona comunidades de usuarios para interacción social.

**Campos**:
| Campo | Tipo de Dato | Propósito | Restricciones | Ejemplo |
|-------|--------------|-----------|--------------|---------|
| `community_id` | INT | Identificador único | PK, AUTO_INCREMENT | 1 |
| `created_by` | INT | ID del usuario creador | NOT NULL, FK (users) | 1 |
| `name` | VARCHAR(50) | Nombre de la comunidad | NOT NULL, UNIQUE | "Ahorro Inteligente" |
| `description` | VARCHAR(255) | Descripción de la comunidad | NULL | "Comunidad para aprender ahorro" |
| `image_url` | VARCHAR(255) | URL de la imagen de la comunidad | NOT NULL | "https://example.com/communities/ahorro.jpg" |
| `creation_date` | TIMESTAMP | Fecha de creación | DEFAULT CURRENT_TIMESTAMP | 2025-05-30 13:42:00 |
| `is_active` | BOOLEAN | Estado activo | NOT NULL, DEFAULT TRUE | TRUE |
| `deleted_at` | TIMESTAMP | Fecha de eliminación lógica | NULL | NULL |
| `created_at` | TIMESTAMP | Fecha de creación | DEFAULT CURRENT_TIMESTAMP | 2025-05-30 13:42:00 |
| `updated_at` | TIMESTAMP | Fecha de última actualización | DEFAULT CURRENT_TIMESTAMP ON UPDATE | 2025-05-30 13:42:00 |
| `updated_by` | INT | ID del usuario que actualizó | NOT NULL, FK (users) | 1 |

### 16. messages
**Descripción**: Almacena mensajes enviados en las comunidades.

**Campos**:
| Campo | Tipo de Dato | Propósito | Restricciones | Ejemplo |
|-------|--------------|-----------|--------------|---------|
| `message_id` | INT | Identificador único | PK, AUTO_INCREMENT | 1 |
| `user_id` | INT | ID del usuario autor | NOT NULL, FK (users) | 1 |
| `community_id` | INT | ID de la comunidad | NOT NULL, FK (communities) | 1 |
| `content` | TEXT | Contenido del mensaje | NOT NULL | "¡Gran consejo sobre ahorros!" |
| `sent_date` | TIMESTAMP | Fecha de envío | DEFAULT CURRENT_TIMESTAMP | 2025-05-30 13:42:00 |
| `is_active` | BOOLEAN | Estado activo | NOT NULL, DEFAULT TRUE | TRUE |
| `deleted_at` | TIMESTAMP | Fecha de eliminación lógica | NULL | NULL |
| `created_at` | TIMESTAMP | Fecha de creación | DEFAULT CURRENT_TIMESTAMP | 2025-05-30 13:42:00 |
| `created_by` | INT | ID del usuario creador | NOT NULL, FK (users) | 1 |
| `updated_at` | TIMESTAMP | Fecha de última actualización | DEFAULT CURRENT_TIMESTAMP ON UPDATE | 2025-05-30 13:42:00 |
| `updated_by` | INT | ID del usuario que actualizó | NOT NULL, FK (users) | 1 |

### 17. community_members
**Descripción**: Registra la membresía de usuarios en comunidades.

**Campos**:
| Campo | Tipo de Dato | Propósito | Restricciones | Ejemplo |
|-------|--------------|-----------|--------------|---------|
| `member_id` | INT | Identificador único | PK, AUTO_INCREMENT | 1 |
| `user_id` | INT | ID del usuario | NOT NULL, FK (users) | 1 |
| `community_id` | INT | ID de la comunidad | NOT NULL, FK (communities) | 1 |
| `join_date` | TIMESTAMP | Fecha de unión | DEFAULT CURRENT_TIMESTAMP | 2025-05-30 13:42:00 |
| `role` | ENUM('Admin', 'Member') | Rol en la comunidad | NOT NULL, DEFAULT 'Member' | "Member" |
| `is_active` | BOOLEAN | Estado activo | NOT NULL, DEFAULT TRUE | TRUE |
| `deleted_at` | TIMESTAMP | Fecha de eliminación lógica | NULL | NULL |
| `created_at` | TIMESTAMP | Fecha de creación | DEFAULT CURRENT_TIMESTAMP | 2025-05-30 13:42:00 |
| `created_by` | INT | ID del usuario creador | NOT NULL, FK (users) | 1 |
| `updated_at` | TIMESTAMP | Fecha de última actualización | DEFAULT CURRENT_TIMESTAMP ON UPDATE | 2025-05-30 13:42:00 |
| `updated_by` | INT | ID del usuario que actualizó | NOT NULL, FK (users) | 1 |

### 18. tags
**Descripción**: Almacena etiquetas para categorizar cursos o comunidades.

**Campos**:
| Campo | Tipo de Dato | Propósito | Restricciones | Ejemplo |
|-------|--------------|-----------|--------------|---------|
| `tag_id` | INT | Identificador único | PK, AUTO_INCREMENT | 1 |
| `name` | VARCHAR(50) | Nombre de la etiqueta | NOT NULL, UNIQUE | "Finanzas" |
| `is_active` | BOOLEAN | Estado activo | NOT NULL, DEFAULT TRUE | TRUE |
| `deleted_at` | TIMESTAMP | Fecha de eliminación lógica | NULL | NULL |
| `created_at` | TIMESTAMP | Fecha de creación | DEFAULT CURRENT_TIMESTAMP | 2025-05-30 13:42:00 |
| `created_by` | INT | ID del usuario creador | NOT NULL, FK (users) | 1 |
| `updated_at` | TIMESTAMP | Fecha de última actualización | DEFAULT CURRENT_TIMESTAMP ON UPDATE | 2025-05-30 13:42:00 |
| `updated_by` | INT | ID del usuario que actualizó | NOT NULL, FK (users) | 1 |

### 19. course_tags
**Descripción**: Relaciona cursos con etiquetas para categorización.

**Campos**:
| Campo | Tipo de Dato | Propósito | Restricciones | Ejemplo |
|-------|--------------|-----------|--------------|---------|
| `course_tag_id` | INT | Identificador único | PK, AUTO_INCREMENT | 1 |
| `course_id` | INT | ID del curso | NOT NULL, FK (courses) | 1 |
| `tag_id` | INT | ID de la etiqueta | NOT NULL, FK (tags) | 1 |
| `is_active` | BOOLEAN | Estado activo | NOT NULL, DEFAULT TRUE | TRUE |
| `deleted_at` | TIMESTAMP | Fecha de eliminación lógica | NULL | NULL |
| `created_at` | TIMESTAMP | Fecha de creación | DEFAULT CURRENT_TIMESTAMP | 2025-05-30 13:42:00 |
| `created_by` | INT | ID del usuario creador | NOT NULL, FK (users) | 1 |
| `updated_at` | TIMESTAMP | Fecha de última actualización | DEFAULT CURRENT_TIMESTAMP ON UPDATE | 2025-05-30 13:42:00 |
| `updated_by` | INT | ID del usuario que actualizó | NOT NULL, FK (users) | 1 |

### 20. favorites
**Descripción**: Permite a los usuarios marcar cursos, lecciones o comunidades como favoritos.

**Campos**:
| Campo | Tipo de Dato | Propósito | Restricciones | Ejemplo |
|-------|--------------|-----------|--------------|---------|
| `favorite_id` | INT | Identificador único | PK, AUTO_INCREMENT | 1 |
| `user_id` | INT | ID del usuario | NOT NULL, FK (users) | 1 |
| `entity_type` | ENUM('Course', 'Lesson', 'Community') | Tipo de entidad | NOT NULL | "Course" |
| `entity_id` | INT | ID de la entidad | NOT NULL | 1 |
| `is_active` | BOOLEAN | Estado activo | NOT NULL, DEFAULT TRUE | TRUE |
| `deleted_at` | TIMESTAMP | Fecha de eliminación lógica | NULL | NULL |
| `created_at` | TIMESTAMP | Fecha de creación | DEFAULT CURRENT_TIMESTAMP | 2025-05-30 13:42:00 |
| `created_by` | INT | ID del usuario creador | NOT NULL, FK (users) | 1 |
| `updated_at` | TIMESTAMP | Fecha de última actualización | DEFAULT CURRENT_TIMESTAMP ON UPDATE | 2025-05-30 13:42:00 |
| `updated_by` | INT | ID del usuario que actualizó | NOT NULL, FK (users) | 1 |

### 21. audit_logs
**Descripción**: Registra cambios (inserciones, actualizaciones, eliminaciones) en las tablas para auditoría.

**Campos**:
| Campo | Tipo de Dato | Propósito | Restricciones | Ejemplo |
|-------|--------------|-----------|--------------|---------|
| `log_id` | INT | Identificador único | PK, AUTO_INCREMENT | 1 |
| `table_name` | VARCHAR(50) | Nombre de la tabla afectada | NOT NULL | "users" |
| `record_id` | INT | ID del registro afectado | NOT NULL | 1 |
| `action` | ENUM('INSERT', 'UPDATE', 'DELETE') | Tipo de acción | NOT NULL | "INSERT" |
| `user_id` | INT | ID del usuario que realizó la acción | NOT NULL, FK (users) | 1 |
| `change_date` | TIMESTAMP | Fecha del cambio | DEFAULT CURRENT_TIMESTAMP | 2025-05-30 13:42:00 |
| `old_data` | JSON | Datos antes del cambio | NULL | {"email": "old@example.com"} |
| `new_data` | JSON | Datos después del cambio | NULL | {"email": "new@example.com"} |

---

## Relaciones entre Tablas

1. **roles ↔ users**:
   - Relación: Uno a muchos (`role_id` en `users` referencia `roles.role_id`).
   - Propósito: Asigna un rol a cada usuario.
   - Restricción: `ON DELETE RESTRICT` evita eliminar roles en uso.

2. **users ↔ subcategories, budgets, transactions, lesson_progress, messages, community_members, favorites**:
   - Relación: Uno a muchos (`user_id` en estas tablas referencia `users.user_id`).
   - Propósito: Vincula datos financieros, progreso y comunidad a usuarios.
   - Restricción: `ON DELETE CASCADE` elimina datos asociados si el usuario es eliminado.

3. **categories ↔ subcategories**:
   - Relación: Uno a muchos (`category_id` en `subcategories` referencia `categories.category_id`).
   - Propósito: Organiza subcategorías bajo categorías.
   - Restricción: `ON DELETE CASCADE` elimina subcategorías si la categoría se elimina.

4. **subcategories ↔ budget_allocations, transactions**:
   - Relación: Uno a muchos (`subcategory_id` en estas tablas referencia `subcategories.subcategory_id`).
   - Propósito: Asocia asignaciones y transacciones a subcategorías.
   - Restricción: `ON DELETE CASCADE`.

5. **budgets ↔ budget_allocations, transactions**:
   - Relación: Uno a muchos (`budget_id` en estas tablas referencia `budgets.budget_id`).
   - Propósito: Vincula asignaciones y transacciones a presupuestos.
   - Restricción: `ON DELETE CASCADE`.

6. **routes ↔ route_courses**:
   - Relación: Uno a muchos (`route_id` en `route_courses` referencia `routes.route_id`).
   - Propósito: Asocia cursos a rutas.
   - Restricción: `ON DELETE CASCADE`.

7. **courses ↔ route_courses, lessons**:
   - Relación: Uno a muchos (`course_id` en estas tablas referencia `courses.course_id`).
   - Propósito: Vincula rutas y lecciones a cursos.
   - Restricción: `ON DELETE CASCADE`.

8. **lessons ↔ questions, lesson_progress**:
   - Relación: Uno a muchos (`lesson_id` en estas tablas referencia `lessons.lesson_id`).
   - Propósito: Asocia preguntas y progreso a lecciones.
   - Restricción: `ON DELETE CASCADE`.

9. **questions ↔ answers**:
   - Relación: Uno a muchos (`question_id` en `answers` referencia `questions.question_id`).
   - Propósito: Vincula respuestas a preguntas.
   - Restricción: `ON DELETE CASCADE`.

10. **communities ↔ messages, community_members**:
    - Relación: Uno a muchos (`community_id` en estas tablas referencia `communities.community_id`).
    - Propósito: Asocia mensajes y miembros a comunidades.
    - Restricción: `ON DELETE CASCADE`.

11. **tags ↔ course_tags**:
    - Relación: Uno a muchos (`tag_id` en `course_tags` referencia `tags.tag_id`).
    - Propósito: Asocia etiquetas a cursos.
    - Restricción: `ON DELETE CASCADE`.

12. **users, courses, lessons, communities ↔ audit_logs**:
    - Relación: Uno a muchos (`user_id` en `audit_logs` referencia `users.user_id`).
    - Propósito: Registra cambios realizados por usuarios.
    - Restricción: `ON DELETE RESTRICT`.

---
