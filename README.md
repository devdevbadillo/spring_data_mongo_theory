# Spring Data MongoDB

## Tabla de Contenido

- [Introducción a MongoDB y Spring Data MongoDB](#introduccion-mongodb-spring-data)
  - [¿Qué es MongoDB?](#que-es-mongodb)
    - [Conceptos clave (Documentos, Colecciones, Bases de Datos)](#conceptos-clave-mongodb)
    - [JSON vs BSON](#json-vs-bson)
    - [Ventajas y Casos de Uso de MongoDB](#ventajas-casos-uso-mongodb)
    - [Modelo de Consistencia](#modelo-consistencia-mongodb)
  - [¿Qué es Spring Data MongoDB?](#que-es-spring-data-mongodb)
    - [Propósito y Arquitectura](#proposito-arquitectura-spring-data-mongodb)
    - [Ventajas de usar Spring Data MongoDB](#ventajas-spring-data-mongodb)
  - [Configuración Inicial del Proyecto](#configuracion-inicial-spring-data-mongodb)
    - [Dependencias (Maven/Gradle)](#dependencias-mongodb)
    - [Configuración de Conexión (`application.properties/yml`)](#configuracion-conexion-mongodb)
    - [Habilitar Spring Data MongoDB](#habilitar-spring-data-mongodb)

- [Mapeo de Documentos (Mapping)](#mapeo-documentos)
  - [Entidades y Colecciones](#entidades-colecciones-mongodb)
    - [`@Document`](#document-annotation)
    - [`@Id` y Generación de IDs](#id-generacion-ids)
    - [`@Field`](#field-annotation)
    - [`@Transient`](#transient-annotation)
  - [Tipos de Datos Soportados](#tipos-datos-soportados-mongodb)
  - [Manejo de Enums](#manejo-enums-mongodb)
  - [Colecciones Embebidas y Referencias](#colecciones-embebidas-referencias)
    - [Documentos Embebidos (Embedded Documents)](#documentos-embebidos)
    - [Referencias (DBRef)](#referencias-dbref)
      - [`@DBRef`](#dbref-annotation)
      - [Estrategias de Fetching (Lazy/Eager)](#estrategias-fetching-dbref)
  - [Mapeo Personalizado](#mapeo-personalizado-mongodb)
    - [`MongoCustomConversions`](#mongocustomconversions)
    - [`MappingMongoConverter`](#mappingmongoconverter)

- [Repositorios de Spring Data MongoDB](#repositorios-spring-data-mongodb)
  - [`MongoRepository`](#mongorepository)
    - [Métodos CRUD Estándar](#metodos-crud-estandard)
    - [Definición de Consultas Derivadas de Métodos (Query Methods)](#consultas-derivadas-metodos)
      - [Convenciones de Nomenclatura](#convenciones-nomenclatura-queries)
      - [Palabras Clave Soportadas](#palabras-clave-soportadas)
  - [`ReactiveMongoRepository` (para WebFlux)](#reactivemongorepository)
    - [Introducción a la Programación Reactiva con MongoDB](#introduccion-reactiva-mongodb)
    - [Casos de Uso](#casos-uso-reactive-mongodb)
  - [`@Query` Anotación](#query-annotation-mongodb)
    - [Consultas JSON Directas](#consultas-json-directas)
    - [Uso de SpEL (Spring Expression Language)](#spel-query-mongodb)
    - [Proyección (`fields`)](#proyeccion-query-mongodb)
    - [Ordenación (`sort`)](#ordenacion-query-mongodb)
  - [Paginación y Ordenación](#paginacion-ordenacion-mongodb)
    - [`Pageable`](#pageable-mongodb)
    - [`Sort`](#sort-mongodb)
  - [Manejo de Errores](#manejo-errores-repositorios)

- [MongoTemplate](#mongotemplate)
  - [¿Cuándo usar `MongoTemplate` vs. Repositorios?](#cuando-usar-mongotemplate-vs-repositorios)
  - [Operaciones Básicas](#operaciones-basicas-mongotemplate)
    - [`save()`, `insert()`, `updateFirst()`, `updateMulti()`, `upsert()`, `remove()`](#metodos-crud-mongotemplate)
  - [Consultas Avanzadas con `Query` y `Criteria`](#consultas-avanzadas-mongotemplate)
    - [`Query` y `Criteria` Builder](#query-criteria-builder)
    - [Operadores de Consulta (`$eq`, `$gt`, `$lt`, `$in`, `$regex`, etc.)](#operadores-consulta-mongodb)
    - [Combinación de Criterios (`and`, `or`, `not`)](#combinacion-criterios-mongodb)
  - [Proyección, Ordenación, Paginación](#proyeccion-ordenacion-paginacion-mongotemplate)
  - [Agregaciones con `Aggregation`](#agregaciones-mongotemplate)
    - [`Aggregation` Pipeline (Stages como `$match`, `$group`, `$project`, `$sort`)](#aggregation-pipeline)
    - [Uso de `AggregationResults`](#aggregationresults)
  - [Colecciones (`collectionName` y `collection()`)](#colecciones-mongotemplate)
  - [Manejo de Errores en `MongoTemplate`](#manejo-errores-mongotemplate)

- [Consultas Reactivas con ReactiveMongoTemplate](#consultas-reactivas-reactivemongotemplate)
  - [Configuración de `ReactiveMongoTemplate`](#configuracion-reactivemongotemplate)
  - [Operaciones CRUD Reactivas](#operaciones-crud-reactivas-mongodb)
  - [Consultas Reactivas con `Query` y `Criteria`](#consultas-reactivas-query-criteria)
  - [Agregaciones Reactivas](#agregaciones-reactivas-mongodb)
  - [Manejo de Backpressure](#manejo-backpressure-mongodb)

- [Características Avanzadas y Optimizaciones](#caracteristicas-avanzadas-optimizaciones)
  - [Índices](#indices-mongodb)
    - [Creación de Índices (`@Indexed`, `@CompoundIndex`)](#creacion-indices-mongodb)
    - [Tipos de Índices (Únicos, Compuestos, Texto, Geoespaciales)](#tipos-indices-mongodb)
    - [Consideraciones de Rendimiento de Índices](#consideraciones-rendimiento-indices)
  - [Validación de Esquema](#validacion-esquema-mongodb)
    - [Validación Nativa de MongoDB](#validacion-nativa-mongodb)
    - [Integración con Bean Validation](#integracion-bean-validation-mongodb)
  - [Auditoría](#auditoria-mongodb)
    - [`@EnableMongoAuditing`](#enablemongoaudiing)
    - [`@CreatedDate`, `@LastModifiedDate`, `@CreatedBy`, `@LastModifiedBy`](#auditoria-annotations)
  - [Ciclo de Vida de los Documentos (Callbacks)](#ciclo-vida-documentos-mongodb)
    - [`@EventListener` con eventos de Spring Data MongoDB](#eventlistener-mongodb)
    - [`@PrePersist`, `@PostPersist`, etc.](#callbacks-annotations)
  - [Manejo de Transacciones (Replica Sets)](#manejo-transacciones-mongodb)
    - [Transacciones Multi-Documento/Multi-Colección](#transacciones-multi-documento)
    - [`@Transactional`](#transactional-mongodb)
    - [Consideraciones y Limitaciones](#limitaciones-transacciones-mongodb)
  - [Integración con Spring Boot Actuator](#integracion-spring-boot-actuator-mongodb)
    - [Health Indicators](#health-indicators-mongodb)
    - [Métricas](#metricas-mongodb)
  - [Seguridad](#seguridad-mongodb)
    - [Autenticación y Autorización en MongoDB](#autenticacion-autorizacion-mongodb)
    - [Encriptación en Tránsito y en Reposo](#encriptacion-mongodb)

- [GridFS (Manejo de Archivos Grandes)](#gridfs)
  - [¿Qué es GridFS?](#que-es-gridfs)
  - [Almacenamiento y Recuperación de Archivos](#almacenamiento-recuperacion-gridfs)
  - [Integración con Spring Data MongoDB](#integracion-gridfs-spring-data)

- [Testing en Spring Data MongoDB](#testing-spring-data-mongodb)
  - [Pruebas Unitarias de Repositorios y Servicios](#pruebas-unitarias-mongodb)
  - [Pruebas de Integración con Bases de Datos en Memoria (ej. Fongo, Flapdoodle Embedded MongoDB)](#pruebas-integracion-mongodb-inmemory)
  - [Testcontainers para MongoDB](#testcontainers-mongodb)
  - [Estrategias de Limpieza de Datos](#limpieza-datos-testing)

- [Mejores Prácticas y Rendimiento](#mejores-practicas-rendimiento-mongodb)
  - [Optimización de Consultas](#optimizacion-consultas-mongodb)
    - [`explain()` para análisis de consultas](#explain-mongodb)
    - [Uso eficiente de índices](#uso-eficiente-indices)
  - [Estrategias de Sharding y Replicación](#sharding-replicacion)
  - [Manejo de Conexiones (Connection Pooling)](#manejo-conexiones-mongodb) 
  - [Gestión de Errores y Retries](#gestion-errores-retries-mongodb)


<a id="introduccion-mongodb-spring-data"></a>
## Introducción a MongoDB y Spring Data MongoDB

<a id="que-es-mongodb"></a>
### ¿Qué es MongoDB?

<a id="conceptos-clave-mongodb"></a>
#### Conceptos clave (Documentos, Colecciones, Bases de Datos)

<a id="json-vs-bson"></a>
#### JSON vs BSON

<a id="ventajas-casos-uso-mongodb"></a>
#### Ventajas y Casos de Uso de MongoDB

<a id="modelo-consistencia-mongodb"></a>
#### Modelo de Consistencia

<a id="que-es-spring-data-mongodb"></a>
### ¿Qué es Spring Data MongoDB?

<a id="proposito-arquitectura-spring-data-mongodb"></a>
#### Propósito y Arquitectura

<a id="ventajas-spring-data-mongodb"></a>
#### Ventajas de usar Spring Data MongoDB

<a id="configuracion-inicial-spring-data-mongodb"></a>
### Configuración Inicial del Proyecto

<a id="dependencias-mongodb"></a>
#### Dependencias (Maven/Gradle)

<a id="configuracion-conexion-mongodb"></a>
#### Configuración de Conexión (`application.properties/yml`)

<a id="habilitar-spring-data-mongodb"></a>
#### Habilitar Spring Data MongoDB

<a id="mapeo-documentos"></a>
## Mapeo de Documentos (Mapping)

<a id="entidades-colecciones-mongodb"></a>
### Entidades y Colecciones

<a id="document-annotation"></a>
#### `@Document`

<a id="id-generacion-ids"></a>
#### `@Id` y Generación de IDs

<a id="field-annotation"></a>
#### `@Field`

<a id="transient-annotation"></a>
#### `@Transient`

<a id="tipos-datos-soportados-mongodb"></a>
### Tipos de Datos Soportados

<a id="manejo-enums-mongodb"></a>
### Manejo de Enums

<a id="colecciones-embebidas-referencias"></a>
### Colecciones Embebidas y Referencias

<a id="documentos-embebidos"></a>
#### Documentos Embebidos (Embedded Documents)

<a id="referencias-dbref"></a>
#### Referencias (DBRef)

<a id="dbref-annotation"></a>
##### `@DBRef`

<a id="estrategias-fetching-dbref"></a>
##### Estrategias de Fetching (Lazy/Eager)

<a id="mapeo-personalizado-mongodb"></a>
### Mapeo Personalizado

<a id="mongocustomconversions"></a>
#### `MongoCustomConversions`

<a id="mappingmongoconverter"></a>
#### `MappingMongoConverter`

<a id="repositorios-spring-data-mongodb"></a>
## Repositorios de Spring Data MongoDB

<a id="mongorepository"></a>
### `MongoRepository`

<a id="metodos-crud-estandard"></a>
#### Métodos CRUD Estándar

<a id="consultas-derivadas-metodos"></a>
#### Definición de Consultas Derivadas de Métodos (Query Methods)

<a id="convenciones-nomenclatura-queries"></a>
##### Convenciones de Nomenclatura

<a id="palabras-clave-soportadas"></a>
##### Palabras Clave Soportadas

<a id="reactivemongorepository"></a>
### `ReactiveMongoRepository` (para WebFlux)

<a id="introduccion-reactiva-mongodb"></a>
#### Introducción a la Programación Reactiva con MongoDB

<a id="casos-uso-reactive-mongodb"></a>
#### Casos de Uso

<a id="query-annotation-mongodb"></a>
### `@Query` Anotación

<a id="consultas-json-directas"></a>
#### Consultas JSON Directas

<a id="spel-query-mongodb"></a>
#### Uso de SpEL (Spring Expression Language)

<a id="proyeccion-query-mongodb"></a>
#### Proyección (`fields`)

<a id="ordenacion-query-mongodb"></a>
#### Ordenación (`sort`)

<a id="paginacion-ordenacion-mongodb"></a>
### Paginación y Ordenación

<a id="pageable-mongodb"></a>
#### `Pageable`

<a id="sort-mongodb"></a>
#### `Sort`

<a id="manejo-errores-repositorios"></a>
### Manejo de Errores

<a id="mongotemplate"></a>
## MongoTemplate

<a id="cuando-usar-mongotemplate-vs-repositorios"></a>
### ¿Cuándo usar `MongoTemplate` vs. Repositorios?

<a id="operaciones-basicas-mongotemplate"></a>
### Operaciones Básicas

<a id="metodos-crud-mongotemplate"></a>
#### `save()`, `insert()`, `updateFirst()`, `updateMulti()`, `upsert()`, `remove()`

<a id="consultas-avanzadas-mongotemplate"></a>
### Consultas Avanzadas con `Query` y `Criteria`

<a id="query-criteria-builder"></a>
#### `Query` y `Criteria` Builder

<a id="operadores-consulta-mongodb"></a>
#### Operadores de Consulta (`$eq`, `$gt`, `$lt`, `$in`, `$regex`, etc.)

<a id="combinacion-criterios-mongodb"></a>
#### Combinación de Criterios (`and`, `or`, `not`)

<a id="proyeccion-ordenacion-paginacion-mongotemplate"></a>
### Proyección, Ordenación, Paginación

<a id="agregaciones-mongotemplate"></a>
### Agregaciones con `Aggregation`

<a id="aggregation-pipeline"></a>
#### `Aggregation` Pipeline (Stages como `$match`, `$group`, `$project`, `$sort`)

<a id="aggregationresults"></a>
#### Uso de `AggregationResults`

<a id="colecciones-mongotemplate"></a>
### Colecciones (`collectionName` y `collection()`)

<a id="manejo-errores-mongotemplate"></a>
### Manejo de Errores en `MongoTemplate`

<a id="consultas-reactivas-reactivemongotemplate"></a>
## Consultas Reactivas con ReactiveMongoTemplate

<a id="configuracion-reactivemongotemplate"></a>
### Configuración de `ReactiveMongoTemplate`

<a id="operaciones-crud-reactivas-mongodb"></a>
### Operaciones CRUD Reactivas

<a id="consultas-reactivas-query-criteria"></a>
### Consultas Reactivas con `Query` y `Criteria`

<a id="agregaciones-reactivas-mongodb"></a>
### Agregaciones Reactivas

<a id="manejo-backpressure-mongodb"></a>
### Manejo de Backpressure

<a id="caracteristicas-avanzadas-optimizaciones"></a>
## Características Avanzadas y Optimizaciones

<a id="indices-mongodb"></a>
### Índices

<a id="creacion-indices-mongodb"></a>
#### Creación de Índices (`@Indexed`, `@CompoundIndex`)

<a id="tipos-indices-mongodb"></a>
#### Tipos de Índices (Únicos, Compuestos, Texto, Geoespaciales)

<a id="consideraciones-rendimiento-indices"></a>
#### Consideraciones de Rendimiento de Índices

<a id="validacion-esquema-mongodb"></a>
### Validación de Esquema

<a id="validacion-nativa-mongodb"></a>
#### Validación Nativa de MongoDB

<a id="integracion-bean-validation-mongodb"></a>
#### Integración con Bean Validation

<a id="auditoria-mongodb"></a>
### Auditoría

<a id="enablemongoaudiing"></a>
#### `@EnableMongoAuditing`

<a id="auditoria-annotations"></a>
#### `@CreatedDate`, `@LastModifiedDate`, `@CreatedBy`, `@LastModifiedBy`

<a id="ciclo-vida-documentos-mongodb"></a>
### Ciclo de Vida de los Documentos (Callbacks)

<a id="eventlistener-mongodb"></a>
#### `@EventListener` con eventos de Spring Data MongoDB

<a id="callbacks-annotations"></a>
#### `@PrePersist`, `@PostPersist`, etc.

<a id="manejo-transacciones-mongodb"></a>
### Manejo de Transacciones (Replica Sets)

<a id="transacciones-multi-documento"></a>
#### Transacciones Multi-Documento/Multi-Colección

<a id="transactional-mongodb"></a>
#### `@Transactional`

<a id="limitaciones-transacciones-mongodb"></a>
#### Consideraciones y Limitaciones

<a id="integracion-spring-boot-actuator-mongodb"></a>
### Integración con Spring Boot Actuator

<a id="health-indicators-mongodb"></a>
#### Health Indicators

<a id="metricas-mongodb"></a>
#### Métricas

<a id="seguridad-mongodb"></a>
### Seguridad

<a id="autenticacion-autorizacion-mongodb"></a>
#### Autenticación y Autorización en MongoDB

<a id="encriptacion-mongodb"></a>
#### Encriptación en Tránsito y en Reposo


<a id="gridfs"></a>
## GridFS (Manejo de Archivos Grandes)

<a id="que-es-gridfs"></a>
### ¿Qué es GridFS?

<a id="almacenamiento-recuperacion-gridfs"></a>
### Almacenamiento y Recuperación de Archivos

<a id="integracion-gridfs-spring-data"></a>
### Integración con Spring Data MongoDB


<a id="testing-spring-data-mongodb"></a>
## Testing en Spring Data MongoDB

<a id="pruebas-unitarias-mongodb"></a>
### Pruebas Unitarias de Repositorios y Servicios

<a id="pruebas-integracion-mongodb-inmemory"></a>
### Pruebas de Integración con Bases de Datos en Memoria (ej. Fongo, Flapdoodle Embedded MongoDB)

<a id="testcontainers-mongodb"></a>
### Testcontainers para MongoDB

<a id="limpieza-datos-testing"></a>
### Estrategias de Limpieza de Datos

<a id="mejores-practicas-rendimiento-mongodb"></a>
## Mejores Prácticas y Rendimiento

<a id="optimizacion-consultas-mongodb"></a>
### Optimización de Consultas

<a id="explain-mongodb"></a>
#### `explain()` para análisis de consultas

<a id="uso-eficiente-indices"></a>
#### Uso eficiente de índices

<a id="sharding-replicacion"></a>
### Estrategias de Sharding y Replicación

<a id="manejo-conexiones-mongodb"></a>
### Manejo de Conexiones (Connection Pooling)

<a id="gestion-errores-retries-mongodb"></a>
### Gestión de Errores y Retries
