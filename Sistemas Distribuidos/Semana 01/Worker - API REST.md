
---

## ¿Qué es un Worker?

Un **worker** (o proceso de fondo) es un programa diseñado para procesar tareas pesadas o repetitivas fuera del flujo principal de la aplicación.

- **Ejecución síncrona:** Trabaja en segundo plano de manera asíncrona.
- **Disparadores:** Se activa mediante colas de mensajes (como RabbitMQ) o eventos programados (cron jobs).
- **Sin interfaz:** No expone rutas web ni endpoints públicos.
- **Caso de uso:** Procesar un video subido por un usuario, enviar correos masivos o generar reportes PDF complejos.

## ¿Qué es una API REST?

Una **API REST** es un servicio web basado en el protocolo HTTP que permite la comunicación directa y estandarizada entre dos sistemas.

- **Comunicación inmediata:** Funciona mediante un modelo de petición-respuesta en tiempo real.
- **Métodos estándar:** Utiliza verbos HTTP como `GET`, `POST`, `PUT` y `DELETE`.
- **Sin estado:** Cada petición del cliente debe contener toda la información necesaria para completarse.
- **Caso de uso:** Consultar el saldo de una cuenta bancaria, iniciar sesión en una app o consultar productos en una tienda online.

### Comparación Directa

|Característica|API REST|Worker|
|---|---|---|
|**Interacción**|Síncrona (Espera/Respuesta rápida)|Asíncrona (Segundo plano)|
|**Iniciador**|Un cliente externo (App, Web)|Una cola de tareas o un reloj (Cron)|
|**Protocolo/Medio**|HTTP (URLs y Endpoints)|Colas de mensajería (Kafka, RabbitMQ)|
|**Enfoque principal**|Baja latencia (Responder rápido)|Alto volumen (Procesar mucha carga)|

### Cómo trabajan juntos

En sistemas modernos, ambos componentes se complementan. Por ejemplo, en una plataforma de streaming:

1. El usuario sube un video a través de la **API REST**.
2. La API guarda el archivo crudo y añade un mensaje a una cola: _"Hay un nuevo video por procesar"_.
3. La API responde de inmediato al usuario: _"Tu video se está procesando"_.
4. El **Worker** detecta el mensaje en la cola de forma interna, toma el archivo, lo comprime a diferentes resoluciones durante varios minutos y actualiza la base de datos al finalizar.

---

En un sistema profesional, no eliges entre uno u otro; los **combinas** para ofrecer la mejor experiencia de usuario. El flujo ideal funciona así:

```
[Cliente / App] ──(1) POST /generar-pdf──> [ PDF-API ] 
      │                                         │ (2) Guarda orden en cola
      │                                         ▼
      │                                    [ Cola (RabbitMQ/Redis) ]
      │                                         │
      │                                         ▼ (3) Toma la tarea y procesa
[ Ver en pantalla ] <──(5) Notificación ─── [ PDF-Worker ] 
                           (Webhook/WS)
```

1. **La API recibe la orden:** El cliente pide un PDF con 10,000 transacciones. La API REST valida los datos y responde en milisegundos: `"ID de tarea: 456. Procesando..."`.
2. **La API delega el trabajo:** La API mete un mensaje en una cola (Redis o RabbitMQ) con los datos necesarios.
3. **El Worker trabaja en la sombra:** El `pdf-worker` (un proceso de Node.js, Python o Go aislado) detecta el mensaje, consulta la base de datos, genera el PDF pesado y lo sube a la nube (ej. AWS S3).
4. **Notificación:** Al terminar, el worker avisa al cliente por WebSockets o guarda el estado como "Listo" para que el usuario lo descargue.

## ¿Por qué NO usar un `pdf-api` para la generación total?

Intentar que una API REST (`pdf-api`) genere el PDF y lo devuelva directamente en la misma respuesta HTTP (`petición -> espera -> descarga`) genera graves problemas técnicos:

1. *Tiempos de espera del navegador (HTTP Timeouts)*

Las pasarelas de pago, balanceadores de carga (como Nginx o Cloudflare) y los navegadores web cortan las conexiones que tardan demasiado (usualmente tras 30 o 60 segundos). Si tu PDF tarda 2 minutos en generarse debido al volumen de datos, la API REST fallará con un error `504 Gateway Timeout`. El worker no tiene límites de tiempo.

2. *Agotamiento de memoria (Memory Bloat)*

Generar un PDF requiere cargar datos en memoria RAM, renderizar fuentes y compilar gráficos. Si 50 usuarios piden un reporte al mismo tiempo a tu `pdf-api`, el servidor web se quedará sin RAM al instante y tu aplicación entera se caerá (Crash). El `pdf-worker` se puede configurar para procesar tareas de una en una o de tres en tres, protegiendo el servidor.

3. *Mala experiencia de usuario (UX congelada)*

En una API pura, el usuario ve la pantalla de carga ("girando") indefinidamente sin saber si el sistema se colgó o sigue trabajando. Con el modelo de worker, la API libera al usuario de inmediato, permitiéndole seguir navegando por la app mientras el PDF se cocina en el fondo.

4. *Capacidad de reintento automática*

Si la base de datos parpadea mientras la API genera el PDF, la petición del usuario muere y este debe iniciar todo el proceso otra vez. Con un worker, si la tarea falla, los sistemas de colas vuelven a meter el trabajo de forma automática (Retry) sin que el usuario tenga que hacer nada.

---