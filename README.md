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

> El free tier de Render "duerme" tras ~15 min sin tráfico. La primera petición después de eso puede tardar hasta 50-60 segundos mientras el servicio despierta — esto es esperado, no un error.

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
