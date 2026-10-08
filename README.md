# go-protobuffers-grpc

Dos servicios **gRPC** en Go con contratos definidos en **Protocol Buffers** y persistencia en PostgreSQL. Gestionan estudiantes, exámenes, preguntas e inscripciones, y cubren los **cuatro tipos de RPC** que ofrece gRPC.

> Proyecto de aprendizaje construido siguiendo el curso de gRPC y Protobuffers en Go de Platzi.

## Servicios y tipos de RPC

**StudentService** (`:5060`)

| RPC | Tipo | Qué hace |
|---|---|---|
| `GetStudent` | Unario | Obtiene un estudiante por id |
| `SetStudent` | Unario | Crea un estudiante |

**TestService** (`:5070`)

| RPC | Tipo | Qué hace |
|---|---|---|
| `GetTest` / `SetTest` | Unario | Consulta o crea un examen |
| `SetQuestions` | Streaming del cliente | El cliente envía un flujo de preguntas y recibe una sola confirmación al cerrar |
| `EnrollStudents` | Streaming del cliente | Inscribe estudiantes a un examen en un flujo |
| `GetStudentsPerTest` | Streaming del servidor | El servidor envía uno a uno los estudiantes inscritos en un examen |
| `TakeTest` | Bidireccional | El servidor envía preguntas mientras el cliente responde, sobre el mismo canal |

`test.proto` importa los mensajes de `student.proto`, de modo que `GetStudentsPerTest` reutiliza el tipo `Student` entre servicios.

## Estructura

```
studentpb/, testpb/   contratos .proto y código Go generado
server/               implementación de los servicios gRPC
repository/           interfaz de persistencia
database/             implementación con PostgreSQL + esquema SQL
student-server/       binario del StudentService
test-server/          binario del TestService
client/               cliente de ejemplo para cada tipo de RPC
```

Los servidores dependen de la interfaz `repository.Repository`, no de PostgreSQL directamente, y registran **reflection** para poder explorarlos con herramientas como `grpcurl`.

## Stack

Go · gRPC · Protocol Buffers · PostgreSQL · Docker

## Cómo correrlo

```bash
# Base de datos con el esquema ya cargado
docker build -t grpc-db ./database
docker run -d -p 54321:5432 -e POSTGRES_PASSWORD=postgres grpc-db

# Servidores (en terminales separadas)
go run ./student-server
go run ./test-server

# Cliente de ejemplo (en client/main.go se elige qué tipo de RPC ejecutar)
go run ./client
```

También se pueden probar con `grpcurl` gracias a reflection:

```bash
grpcurl -plaintext localhost:5070 list
grpcurl -plaintext -d '{"id":"t1"}' localhost:5070 test.TestService/GetTest
```

## Qué mejoraría para producción

- `TakeTest` usa un examen fijo (`t1`); el id debería llegar en el primer mensaje del flujo.
- Los errores de lectura del stream usan `log.Fatalf`, que tumba el servidor completo; deberían devolver un status gRPC al cliente.
- Configuración por variables de entorno en lugar de la cadena de conexión fija, más TLS, tests e interceptores para logging y métricas.
