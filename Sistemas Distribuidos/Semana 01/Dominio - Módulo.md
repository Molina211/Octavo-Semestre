
---

## Dominio: El "Qué" (El Negocio)

El dominio es la esfera de conocimiento, actividad o negocio para la cual se desarrolla el software. Define la lógica del mundo real, las reglas de negocio y los conceptos que la aplicación debe resolver.

- **Enfoque:** Conceptual, lógico y orientado al negocio.

- **Independencia:** Es independiente de la tecnología (no le importa si usas bases de datos SQL o NoSQL).

- **Ejemplo:** En un sistema de comercio electrónico, el "Dominio" incluye las reglas de cómo se calcula un impuesto, qué es un cliente, o cómo se procesa un reembolso.

- **Subdominios:** Se suele dividir en Core (el núcleo del negocio), Supporting (soporte) y Generic (genérico).

## Módulo: El "Cómo" (La Estructura)

Un módulo es una unidad física de organización de código. Es un componente de software empaquetado que agrupa funciones, clases o métodos relacionados para ser reutilizados o mantener el orden.

- **Enfoque:** Técnico, estructural y orientado a la implementación.

- **Independencia:** Depende del lenguaje o framework (un módulo de Node.js, un package en Java, un módulo en Python).

- **Ejemplo:** Un módulo de autenticación, un módulo de conexión a la base de datos, o un módulo para exportar archivos PDF.

- **Propósito:** Encapsular código, reducir el acoplamiento y facilitar el mantenimiento.

---

### Tabla Comparativa

|Característica|Dominio|Módulo|
|---|---|---|
|**Origen**|Viene del problema del cliente.|Viene de la solución del programador.|
|**Lenguaje**|Usa términos del negocio (Ubiquitous Language).|Usa términos técnicos (clases, funciones, librerías).|
|**Cambios**|Cambia si cambian las leyes o reglas del negocio.|Cambia si se refactoriza o mejora la tecnología.|
|**Representación**|Modelos, diagramas de flujo, entidades lógicas.|Carpetas, archivos, paquetes, librerías `.jar` o `.npm`.|

### ¿Cómo se relacionan?

No son excluyentes; de hecho, coexisten. En arquitecturas modernas (como _Domain-Driven Design_ o Arquitectura Limpia):

- Un **Dominio** puede estar compuesto por varios **Módulos** técnicos.

- Un **Módulo** de software puede ser creado específicamente para contener y proteger la lógica de un **Dominio**.