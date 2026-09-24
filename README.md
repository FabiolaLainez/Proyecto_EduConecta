# EduConecta — API de Tutorías en Línea

Proyecto grupal — Desarrollo de Aplicaciones de Vanguardia, Sección 574
Equipo 5 — Caso 5: EduConecta

## Arquitectura

- **Despliegue:** (N-CAPAS)
- **Organización interna:** N-Capas (Domain / Application / Infrastructure / Api)
- **Base de datos:** MongoDB Community Server, auto-hospedado vía Docker (sin MongoDB Atlas)
- **PaaS:** Render (free tier), API y MongoDB corriendo en el mismo contenedor

El razonamiento completo de estas decisiones está en `Primer_Avance_MongoDB.docx`.

## API en producción

```
https://educonecta-api.onrender.com
```

> El free tier de Render "duerme" tras 15 min sin tráfico. La primera petición después de eso puede tardar hasta 50-60 segundos mientras el servicio despierta — esto es esperado, no un error.

## Endpoints

| Método | Ruta | Descripción |
|---|---|---|
| POST | `/api/tutores` | Registrar un tutor con sus materias |
| GET | `/api/tutores?materia={materia}` | Buscar tutores por materia |
| GET | `/api/tutores/todos` | Listar todos los tutores |
| DELETE | `/api/tutores/{id}` | Eliminar un tutor (falla si tiene sesiones futuras) |
| POST | `/api/estudiantes` | Registrar un estudiante |
| GET | `/api/estudiantes/todos` | Listar todos los estudiantes |
| GET | `/api/estudiantes/{id}/sesiones` | Listar las sesiones de un estudiante |
| POST | `/api/sesiones` | Agendar una sesión de tutoría |
| PATCH | `/api/sesiones/{id}/completar` | Marcar una sesión como completada (falla si aún no ha pasado) |
| GET | `/api/sesiones/todas` | Listar todas las sesiones |
| GET | `/api/health/mongo` | Verificar la conexión a MongoDB |

## Reglas de negocio implementadas

1. Un tutor no puede tener dos sesiones agendadas en el mismo horario.
2. Una sesión no puede agendarse con una materia que el tutor no imparte.
3. La duración de una sesión debe estar entre 30 y 180 minutos.
4. No se puede marcar como completada una sesión cuya fecha/hora todavía no ha pasado.
5. Un estudiante no puede agendar dos sesiones en el mismo horario, aunque sean con tutores distintos.
6. No se puede eliminar un tutor que tiene sesiones futuras agendadas.

Todas están implementadas como validadores en `EduConecta.Domain/Validation/SesionValidator.cs`, no como comentarios.

## Cómo probar con Postman

1. Importar la colección y el environment
2. Abre Postman.
3. *Import* → arrastra o selecciona EduConecta.postman_collection.json.
4. Repite *Import* con EduConecta-Render.postman_environment.json.
5. Arriba a la derecha, en el selector de Environment, elige *"EduConecta - Render"*.

### Correr las peticiones individualmente

Con el environment activo, abre cualquier petición de la colección y dale *Send*. Cada una ya usa {{base_url}} (apunta directo a producción) y las variables encadenadas ({{tutor_id}}, {{estudiante_id}}, {{sesion_id}}, {{sesion_pasada_id}}) que se van llenando automáticamente conforme corres las peticiones que las generan.

*Orden recomendado* (algunas dependen de las anteriores):
1. POST Tutor
2. GET Buscar por materia
3. POST estudiantes
4. POST sesiones
5. POST Sesión duración inválida 
6. PATCH Completar futura 
7. POST Sesión pasada
8. PATCH Completar pasada
9. GET Sesiones estudiante id
10. DELETE Tutor con sesión futura 

### Correr toda la colección de una vez 

1. Click derecho sobre *My Collection* → *Run collection*.
2. Selecciona el environment *"EduConecta - Render"*.
3. Click *Run*.
4. Postman ejecuta las 12 peticiones en orden y muestra un resumen con el resultado de cada pm.test.

## Correr el proyecto en local (opcional)
Requisitos: .NET 8.0 SDK, Docker Desktop.

bash
# Levantar MongoDB en Docker
docker run -d --name educonecta-mongo -p 27017:27017 -v educonecta-mongo-data:/data/db mongo:7

# Correr la API
dotnet run --project src/EduConecta.Api

La API queda disponible en http://localhost:5044 o el puerto que indique la consola.

## Equipo
- Luis Eduardo López
- Anton Leonardo Acosta
- Fabiola Michelle Lainez
