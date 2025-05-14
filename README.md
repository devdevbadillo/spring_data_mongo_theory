# Hibernate

## Tabla de Contenido

- [Fundamentos y arquitectura de Hibernate](#fundamentos-de-hibernate)
  - [¿Qué es Hibernate y el problema de la impedancia objeto-relacional?](#que-es-hibernate)
    - [Arquitectura de Hibernate](#interfaces-centrales-en-jpa)
        - [El componente SessionFactory](#componente-session-factory)
        - [El componente Session](#componente-session)
            - [Ciclo de Vida de la Sesión](#ciclo-de-vida-de-la-session)
        - [El componente Transaction](#componente-transaction)
  - [Configuración programática de Hibernate](#configuracion-de-hibernate)
- [Mapeo Objeto-Relacional (ORM)](#orm)
    - [Mapeo con anotaciones](#mapeo-con-anotaciones)
      - [Anotaciones básicas](#anotaciones-basicas)
      - [Mapeo de asociaciones](#mapeo-de-asociaones)
    - [Herencia](#herencia)
      - [Tabla por clase (Table per Class)](#tabla-por-clase)
      - [Tabla por subclase (Table per Subclass)](#tabla-por-subclase)
      - [Tabla única por jerarquía (Single Table per Hierarchy)](#tabla-unica-por-jerarquia)
- [Persitencia y optimización](#persistencia-y-optimizacion)
    - [Operaciones CRUD](#operaciones-crud)
        - [Uso del Session para operaciones básicas](#uso-de-session-en-crud)
    - [Caché de Hibernate](#cache-de-hibernate)
        - [Caché de primer nivel](#cache-de-primer-nivel)
        - [Caché de segundo nivel](#cache-de-segundo-nivel)
        - [Otros niveles de caché](#otros-niveles-de-caches)
    - [Fetch Strategies (Estrategias de Carga)](#estrategias-de-carga)
- [Consultas y recuperación de datos](#consultas-y-recuperacion-de-datos)
    - [HQL (Hibernate Query Language)](#HQL)
    - [Criteria API](#criteria-api)
    - [Native SQL](#native-sql)
    - [Proyecciones y agregaciones](#proyecciones-y-agregaciones)
- [Temas avanzados](#temas-avanzados)
    - [Interceptores y listeners](#interceptores-y-listeners)
    - [Generación de esquema](#generacion-de-esquema)
 

<a id="fundamentos-de-hibernate"></a>
## Fundamentos y arquitectura de Hibernate

<a id="que-es-hibernate"></a>
### ¿Qué es Hibernate y el problema de la impedancia objeto-relacional?
Imaginamos que se está construyendo una aplicación Java para gestionar una biblioteca. En el código Java, representas los libros, autores y socios como **objetos**, con sus atributos (título, nombre, dirección) y comportamientos (prestar libro, devolver libro). Por otro lado, la información de la biblioteca necesita ser almacenada de forma persistente para que no se pierda cuando la aplicación se apaga. Tradicionalmente, esto se hace en una base de datos relacional, **organizada en tablas con filas y columnas**.

Aquí es donde surge el problema de la impedancia objeto-relacional (Object-Relational Impedance Mismatch). Las diferencias fundamentales entre **cómo modelamos los datos en el mundo orientado a objetos y cómo se estructuran en las bases de datos relacionales** generan una serie de desafíos:

1. Granularidad: Los objetos pueden tener relaciones complejas y anidadas, mientras que las bases de datos relacionales tienden a tener estructuras más planas.
   
2. Subtipos: La herencia es un concepto fundamental en la programación orientada a objetos, pero no tiene una correspondencia directa en el modelo relacional.

3. Identidad: Los objetos tienen una identidad basada en su instancia en memoria, mientras que las filas en una tabla se identifican principalmente por su clave primaria.

4. Asociaciones: Las relaciones entre objetos (referencias) se manejan de manera diferente a las relaciones entre tablas (claves foráneas).

5. Manipulación de Datos: En Java, trabajamos con objetos y sus métodos. Para interactuar con la base de datos, necesitamos escribir código SQL, que es un lenguaje diferente y se centra en la manipulación de conjuntos de datos.

Hibernate entra en escena como una **solución a este problema**. Es un framework de Mapeo Objeto-Relacional (ORM) para la plataforma Java. Su objetivo principal es automatizar la persistencia de objetos Java en bases de datos relacionales.

> [!NOTE]
>
> Hibernate actúa como una capa intermedia que traduce las operaciones realizadas en los objetos Java a operaciones equivalentes en la base de datos, y viceversa.

Maneja la traducción entre el modelo de objetos y el modelo relacional, facilitando el desarrollo de aplicaciones persistentes y reduciendo la cantidad de código repetitivo (boilerplate) que se tendría que escribir para realizar las operaciones de base de datos.

<a id="interfaces-centrales-en-jpa"></a>
#### Arquitectura de Hibernate
La arquitectura de Hibernate se basa en varios componentes clave que trabajan juntos para lograr la persistencia de los objetos. 

> Componentes Principales

1. SessionFactory (Fábrica de Sesiones):

> - Se crea una vez durante el inicio de la aplicación.
> - Contiene la configuración de la base de datos y los metadatos de mapeo (cómo las clases Java se relacionan con las tablas de la base de datos).
> - Actúa como una fábrica para las instancias de Session.
> - Es thread-safe, lo que significa que múltiples hilos pueden acceder a ella simultáneamente sin problemas de concurrencia.

2. Session (Sesión):

> - Representa una conexión lógica con la base de datos.
> - Es un objeto ligero y no thread-safe, por lo que **se debe obtener una nueva instancia para cada unidad de trabajo** (por ejemplo, una petición web o una transacción).
> - Proporciona los métodos para realizar las operaciones CRUD (Crear, Leer, Actualizar, Borrar) sobre los objetos persistentes.
> - Mantiene una caché de primer nivel (caché a nivel de sesión) de los objetos que ha cargado o guardado durante la transacción actual.
> - **Es la interfaz principal para interactuar con Hibernate**.


3. Transaction (Transacción):

> - Representa una unidad de trabajo atómica.
> - **Encapsula un conjunto de operaciones** de base de datos que deben completarse con éxito o fallar como una sola unidad.
> - Hibernate abstrae el concepto de transacción, permitiéndote trabajar con transacciones independientemente del sistema subyacente (JDBC o JTA).
> - La Session tiene un objeto Transaction asociado. Debes iniciar, confirmar (commit), o deshacer (rollback) la transacción.

4. Metadatos de Mapeo:

> - Definen cómo las clases Java se mapean a las tablas de la base de datos.
> - Pueden definirse a través de archivos XML (.hbm.xml) o mediante anotaciones JPA directamente en las clases Java.
> - La SessionFactory utiliza estos metadatos para **entender cómo persistir y recuperar los objetos**.

5. Dialecto:

> - Un componente específico de Hibernate que conoce las particularidades de la base de datos que se está utilizando (por ejemplo, MySQL, PostgreSQL, Oracle).
> - Permite a Hibernate generar el SQL optimizado para la base de datos específica, manejar las diferencias en los tipos de datos y las funciones SQL.


<a id="componente-session-factory"></a>
##### El componente SessionFactory
Como se ha mencionado, la SessionFactory es una pieza central en la arquitectura de Hibernate. La SessionFactory se crea una sola vez durante el inicio de la aplicación, es un proceso costoso porque Hibernate lee los archivos de mapeo, configura la conexión a la base de datos y construye la infraestructura interna.

Una vez creada, **la SessionFactory es inmutable**. Si se necesita cambiar la configuración (por ejemplo, conectarse a otra base de datos), se debe de crear una nueva instancia de SessionFactory. En la mayoría de las aplicaciones, se recomienda tener una única instancia de SessionFactory para toda la aplicación. Esto se puede lograr mediante un patrón Singleton o utilizando mecanismos de inyección de dependencias (como en Spring).


<a id="componente-session"></a>
##### El componente Session
La Session es la interfaz principal a través de la cual tu aplicación interactúa con Hibernate para realizar operaciones de persistencia. Se obtiene una nueva instancia de Session a partir de la SessionFactory para cada unidad de trabajo.

Proporciona métodos como save(), get(), update(), delete(), merge(), persist(), etc., para manipular los objetos persistentes. Además, mantiene un registro de los objetos que han sido cargados o guardados dentro de la sesión actual. Si se intenta cargar el mismo objeto varias veces dentro de la misma sesión, Hibernate generalmente lo recuperará de la caché en lugar de ir a la base de datos.

> [!NOTE]
> Permite crear y ejecutar consultas utilizando HQL (Hibernate Query Language), Criteria API o SQL nativo.
> Proporciona acceso al objeto Transaction asociado a la sesión para iniciar, confirmar o deshacer transacciones. 

<a id="ciclo-de-vida-de-la-session"></a>
###### Ciclo de Vida de la Sesión
El ciclo de vida de una instancia de Session es relativamente corto y está ligado a una unidad de trabajo. Generalmente sigue estos pasos:

1. Apertura: Se obtiene una nueva instancia de Session a partir de la SessionFactory al inicio de una unidad de trabajo (por ejemplo, al recibir una petición web o al iniciar un proceso por lotes).

2. Trabajo: Se realizan las operaciones de persistencia necesarias: cargar, guardar, actualizar, borrar objetos, ejecutar consultas. Durante este tiempo, la Session gestiona la caché de primer nivel y registra los cambios realizados en los objetos persistentes.

3. Flush (Opcional pero Común): En algún punto durante la unidad de trabajo, o antes de confirmar la transacción, se puede llamar al método flush() para sincronizar los cambios en la caché con la base de datos.

4. Confirmación o Deshacer (Commit o Rollback): Si todas las operaciones dentro de la unidad de trabajo fueron exitosas, se confirma la transacción (commit). Si hubo algún error, se deshace la transacción (rollback), lo que revierte todos los cambios realizados en la base de datos dentro de esa transacción.

5. Cierre: Una vez que la transacción se ha confirmado o deshecho, y todas las operaciones necesarias se han completado, es crucial cerrar la Session utilizando el método close(). Esto libera los recursos asociados a la sesión, incluyendo la conexión a la base de datos (que puede ser devuelta a un pool de conexiones).

> Estados de los objetos persistentes dentro del ciclo de vida de la sesión
> 
> - Transient (Transitorio): Un objeto que no está asociado a ninguna Session y no tiene una representación en la base de datos. Acabas de crearlo con el operador new.
>
> - Persistent (Persistente): Un objeto que está asociado a una Session y tiene una representación en la base de datos. Ha sido guardado, cargado o creado dentro de la sesión actual. Cualquier cambio realizado en un objeto persistente dentro de una transacción activa se detectará y se sincronizará con la base de datos (auto-dirty checking).
>
> - Detached (Desvinculado): Un objeto que previamente fue persistente pero la Session que lo gestionaba se ha cerrado. El objeto todavía tiene su identificador, pero ya no está bajo la supervisión de ninguna Session. Para volver a hacerlo persistente, necesitas asociarlo a una nueva Session utilizando métodos como merge() o update().

<a id="componente-transaction"></a>
##### El componente Transaction
Hibernate proporciona **su propia interfaz Transaction**, que abstrae las APIs de transacción subyacentes (JDBC o JTA). Esto hace que el código sea más portable. La Transaction define los límites de una unidad lógica de trabajo. Todas las operaciones de base de datos realizadas dentro de una transacción deben tratarse como una sola unidad.

> [!IMPORTANT
> Cada instancia de Session tiene un objeto Transaction asociado (que se puede obtener con session.getTransaction())

- Operaciones Principales:
1. begin(): Inicia una nueva transacción.
2. commit(): Confirma los cambios realizados dentro de la transacción y los hace permanentes en la base de datos.
3. rollback(): Deshace todos los cambios realizados dentro de la transacción, revirtiendo la base de datos a su estado anterior al inicio de la transacción.
4. isActive(): Verifica si la transacción está activa.
5. setTimeout(): Establece un tiempo de espera para la transacción.

<a id="configuracion-de-hibernate"></a>
### Configuración programática de Hibernate
Hibernate también se puede configurar programáticamente utilizando la API de configuración. Esto permite más control sobre cómo se inicializa Hibernate.

> Pasos para la configuración programática

1. Crear una Instancia de Configuration
```
import org.hibernate.cfg.Configuration;

Configuration configuration = new Configuration();
```

2. Configurar las pr1opiedades de Hibernate: Se pueden establecer las propiedades directamente en el objeto Configuration utilizando el método setProperty()
```
configuration.setProperty("hibernate.connection.driver_class", "com.mysql.cj.jdbc.Driver");
configuration.setProperty("hibernate.connection.url", "jdbc:mysql://localhost:3306/midatabase");
configuration.setProperty("hibernate.connection.username", "miusuario");
configuration.setProperty("hibernate.connection.password", "micontraseña");
configuration.setProperty("hibernate.dialect", "org.hibernate.dialect.MySQLDialect");
configuration.setProperty("hibernate.hbm2ddl.auto", "update"); // Para la gestión automática del esquema
configuration.setProperty("hibernate.show_sql", "true"); // Para ver las consultas SQL generadas
```

3. Agregar las clases mapeadas (Entidades): Es necesario indicarle a Hibernate qué clases Java deben ser mapeadas a las tablas de la base de datos.
```
configuration.addAnnotatedClass(com.example.modelo.Libro.class);
configuration.addAnnotatedClass(com.example.modelo.Autor.class);
// Si se utilizan archivos XML de mapeo:
// configuration.addResource("com/example/mapeo/Libro.hbm.xml");
// configuration.addResource("com/example/mapeo/Autor.hbm.xml");
```

4. Construir la SessionFactory
```
import org.hibernate.SessionFactory;
import org.hibernate.boot.registry.StandardServiceRegistryBuilder;
import org.hibernate.service.ServiceRegistry;

ServiceRegistry serviceRegistry = new StandardServiceRegistryBuilder()
        .applySettings(configuration.getProperties()).build();
SessionFactory sessionFactory = configuration.buildSessionFactory(serviceRegistry);
```

<a id="orm"></a>
## Mapeo Objeto-Relacional (ORM)
Mapeo Objeto-Relacional (ORM) es la técnica de mapear las estructuras de datos de una aplicación orientada a objetos al esquema de una base de datos relacional. Un framework ORM como Hibernate se encarga de esta traducción de forma automática, permitiendo interactuar con la base de datos utilizando tus objetos Java en lugar de escribir directamente código SQL.

<a id="mapeo-con-anotaciones"></a>
### Mapeo con anotaciones
El mapeo con anotaciones se basa en el uso de anotaciones JPA (Java Persistence API) directamente en las clases Java para especificar cómo se deben mapear al esquema de la base de datos. Hibernate, como implementación de JPA, soporta estas anotaciones. 

<a id="anotaciones-basicas"></a>
#### Anotaciones básicas

1. @Entity: Marca una clase Java como una entidad persistente. Esto indica a Hibernate que las instancias de esta clase deben ser mapeadas a una tabla en la base de datos.
```
import jakarta.persistence.Entity;

@Entity(name="productos") 
public class Producto {
    // ... atributos y métodos ...
}
```

2. @Table: Se utiliza a nivel de clase para especificar detalles sobre la tabla de la base de datos a la que se mapea la entidad. Se puede definir el nombre de la tabla, el esquema, los constraints únicos, etc.
```
import jakarta.persistence.Table;

@Entity
@Table(name = "TBL_PRODUCTOS", schema = "inventario", uniqueConstraints = {
        @jakarta.persistence.UniqueConstraint(columnNames = {"codigo"})
})
public class Producto {
    // ...
}
```

3. @Id: Marca un atributo de la clase como la clave primaria de la entidad.

```
import jakarta.persistence.Id;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.GenerationType;

@Entity
public class Producto {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    // ...
}
```
@GeneratedValue: Especifica la estrategia de generación de la clave primaria. Los valores comunes para el atributo strategy incluyen:
> - GenerationType.AUTO: Hibernate elige una estrategia apropiada basada en la base de datos.
> - GenerationType.IDENTITY: La base de datos genera el valor (por ejemplo, mediante autoincremento en MySQL).
> - GenerationType.SEQUENCE: Utiliza una secuencia de base de datos para generar los valores.
> - GenerationType.TABLE: Utiliza una tabla especial para simular una secuencia.

4. @Column: Se utiliza para mapear un atributo de la clase a una columna específica de la tabla. Se puede especificar el nombre de la columna, si permite valores nulos (nullable), si es único (unique), la longitud (length), el tipo (columnDefinition), etc.
```
import jakarta.persistence.Column;

@Entity
public class Producto {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(name = "nombre_producto", nullable = false, length = 100)
    private String nombre;

    @Column(precision = 10, scale = 2)
    private java.math.BigDecimal precio;

    // ...
}
```

5. @Basic: Es la anotación por defecto para los atributos de tipo primitivo o wrappers y String. Se utiliza para especificar opciones de carga (eager o lazy), aunque generalmente no es necesario especificarla explícitamente.
```
import jakarta.persistence.Basic;
import jakarta.persistence.FetchType;

@Entity
public class Producto {
    // ...

    @Basic(fetch = FetchType.LAZY)
    private String descripcion;

    // ...
}
```

<a id="mapeo-de-asociaones"></a>
#### Mapeo de asociaciones
Uno de los aspectos más importantes y a veces complejos del ORM es el mapeo de las relaciones entre las entidades. JPA proporciona varias anotaciones para definir estas asociaciones:

1. OneToOne (Uno a Uno): Representa una relación donde una instancia de una entidad está relacionada con exactamente una instancia de otra entidad.
```
@Entity
public class Persona {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String nombre;

    @OneToOne(cascade = CascadeType.ALL)
    @JoinColumn(name = "direccion_id", unique = true)
    private Direccion direccion;

    // ...
}

@Entity
public class Direccion {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String calle;
    private String ciudad;

    @OneToOne(mappedBy = "direccion")
    private Persona persona; // Para la relación bidireccional

    // ...
}
```
  * `@JoinColumn(name = "direccion_id", unique = true)`: Especifica la columna de clave foránea en la tabla `Persona` que referencia la tabla `Direccion`. `unique = true` asegura la cardinalidad uno a uno.
  * `mappedBy = "direccion"`: En la entidad `Direccion`, indica que la relación es bidireccional y que la entidad propietaria de la relación es `Persona` a través de su atributo `direccion`.

2. @ManyToOne (Muchos a Uno): Representa una relación donde muchas instancias de una entidad pueden estar relacionadas con una instancia de otra entidad.
```
@Entity
public class Pedido {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private java.util.Date fechaPedido;

    @ManyToOne
    @JoinColumn(name = "cliente_id", nullable = false)
    private Cliente cliente;

    // ...
}

@Entity
public class Cliente {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String nombre;

    @OneToMany(mappedBy = "cliente", cascade = CascadeType.ALL)
    private java.util.List<Pedido> pedidos; // Para la relación bidireccional

    // ...
}
```

 * `@JoinColumn(name = "cliente_id", nullable = false)`: Especifica la columna de clave foránea en la tabla `Pedido` que referencia la tabla `Cliente`. `nullable = false` indica que cada pedido debe tener un cliente asociado.
 * `@OneToMany(mappedBy = "cliente", cascade = CascadeType.ALL)`: En la entidad `Cliente`, define la relación inversa (uno a muchos). `mappedBy` indica el atributo en la entidad `Pedido` que mapea esta relación.

3. @ManyToMany (Muchos a Muchos): Representa una relación donde muchas instancias de una entidad pueden estar relacionadas con muchas instancias de otra entidad. Esta relación generalmente se implementa mediante una tabla de unión en la base de datos.

```
@Entity
public class Estudiante {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String nombre;

    @ManyToMany(cascade = { CascadeType.PERSIST, CascadeType.MERGE })
    @JoinTable(
        name = "estudiante_curso",
        joinColumns = { @JoinColumn(name = "estudiante_id") },
        inverseJoinColumns = { @JoinColumn(name = "curso_id") }
    )
    private java.util.Set<Curso> cursos;

    // ...
}

@Entity
public class Curso {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String titulo;

    @ManyToMany(mappedBy = "cursos")
    private java.util.Set<Estudiante> estudiantes; // Para la relación bidireccional

    // ...
}
```

 * `@JoinTable `: Especifica los detalles de la tabla de unión (estudiante_curso).
 * `joinColumns `: Define la columna de clave foránea en la tabla de unión que referencia la tabla Estudiante (estudiante_id).
 * `inverseJoinColumns `: Define la columna de clave foránea en la tabla de unión que referencia la tabla Curso (curso_id).
 * `mappedBy ` = "cursos": En la entidad Curso, indica el atributo en la entidad Estudiante que mapea esta relación.

<a id="herencia"></a>
### Herencia
JPA y Hibernate proporcionan varias estrategias para mapear jerarquías de clases a tablas en la base de datos.


<a id="tabla-por-clase"></a>
#### Tabla por clase (Table per Class)
En esta estrategia, **cada clase concreta en la jerarquía se mapea a su propia tabla**. La tabla de cada subclase contiene todas las columnas de la superclase más las columnas específicas de la subclase.

> Ejemplo
```
@Entity
@Inheritance(strategy = InheritanceType.TABLE_PER_CLASS)
public abstract class Vehiculo {
    @Id
    @GeneratedValue(strategy = GenerationType.TABLE)
    private Long id;

    private String marca;

    // ...
}

@Entity
public class Coche extends Vehiculo {
    private int numeroPuertas;

    // ...
}

@Entity
public class Moto extends Vehiculo {
    private boolean tieneSidecar;

    // ...
}
```

 * `@Inheritance(strategy = InheritanceType.TABLE_PER_CLASS)`: Se utiliza en la clase base de la jerarquía para indicar la estrategia de herencia.
 * `@GeneratedValue(strategy = GenerationType.TABLE)`: Es importante utilizar TABLE como estrategia de generación de ID para evitar conflictos entre las tablas de las subclases. Hibernate utiliza una tabla separada para gestionar los IDs de toda la jerarquía.
  
<a id="tabla-por-subclase"></a>
#### Tabla por subclase (Table per Subclass)
En esta estrategia, **cada clase concreta se mapea a su propia tabla, que contiene solo las columnas específicas de esa subclase, más una columna de clave primaria que también es clave foránea a la tabla de la superclase**. La tabla de la superclase contiene las columnas comunes a todas las subclases.

```
@Entity
@Inheritance(strategy = InheritanceType.JOINED)
public abstract class Empleado {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String nombre;
    private java.util.Date fechaContratacion;

    // ...
}

@Entity
@Table(name = "empleado_salariado")
public class EmpleadoSalariado extends Empleado {
    private java.math.BigDecimal salario;

    // ...
}

@Entity
@Table(name = "empleado_por_hora")
public class EmpleadoPorHora extends Empleado {
    private java.math.BigDecimal tarifaPorHora;
    private int horasTrabajadas;

    // ...
}
```
* `@Inheritance(strategy = InheritanceType.JOINED)`: Indica la estrategia de herencia de tabla por subclase.
* Cada subclase tiene su propia tabla `(empleado_salariado, empleado_por_hora)` con sus atributos específicos y una clave foránea que referencia la clave primaria de la tabla empleado

<a id="tabla-unica-por-jerarquia"></a>
#### Tabla única por jerarquía (Single Table per Hierarchy)
En esta estrategia, **toda la jerarquía de clases se mapea a una única tabla**. Esta tabla contiene columnas para todos los atributos de la superclase y de todas las subclases. Se utiliza una columna discriminadora para identificar el tipo de cada fila (es decir, a qué subclase corresponde). Las columnas que no son aplicables a una fila específica contendrán valores nulos.

> Ejemplo
```
@Entity
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
@DiscriminatorColumn(name = "tipo_empleado", discriminatorType = DiscriminatorType.STRING)
@DiscriminatorValue("EMPLEADO")
public abstract class Empleado {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String nombre;
    private java.util.Date fechaContratacion;

    // ...
}

@Entity
@DiscriminatorValue("SALARIADO")
public class EmpleadoSalariado extends Empleado {
    private java.math.BigDecimal salario;

    // ...
}

@Entity
@DiscriminatorValue("POR_HORA")
public class EmpleadoPorHora extends Empleado {
    private java.math.BigDecimal tarifaPorHora;
    private int horasTrabajadas;

    // ...
}
```
* `Inheritance(strategy = InheritanceType.SINGLE_TABLE)`: Indica la estrategia de tabla única por jerarquía.
* `@DiscriminatorColumn(name = "tipo_empleado", discriminatorType = DiscriminatorType.STRING)`: Define la columna discriminadora (tipo_empleado) y su tipo de datos (STRING).
* `@DiscriminatorValue("EMPLEADO"), @DiscriminatorValue("SALARIADO"), @DiscriminatorValue("POR_HORA")`: Especifican el valor que se almacenará en la columna discriminadora para cada tipo de entidad.

<a id="persistencia-y-optimizacion"></a>
## Persitencia y optimización

<a id="operaciones-crud"></a>
### Operaciones CRUD

<a id="uso-de-session-en-crud"></a>
#### Uso del Session para operaciones básicas

<a id="cache-de-hibernate"></a>
### Caché de Hibernate

<a id="cache-de-primer-nivel"></a>
#### Caché de primer nivel

<a id="cache-de-segundo-nivel"></a>
#### Caché de segundo nivel

<a id="otros-niveles-de-caches"></a>
#### Otros niveles de caché

<a id="estrategias-de-carga"></a>
### Fetch Strategies (Estrategias de Carga)

<a id="consultas-y-recuperacion-de-datos"></a>
## Consultas y recuperación de datos

<a id="HQL"></a>
### HQL (Hibernate Query Language)

<a id="criteria-api"></a>
### Criteria API

<a id="native-sql"></a>
### Native SQL

<a id="proyecciones-y-agregaciones"></a>
### Proyecciones y agregaciones

<a id="temas-avanzados"></a>
## Temas avanzados

<a id="interceptores-y-listeners"></a>
### Interceptores y listeners

<a id="generacion-de-esquema"></a>
### Generación de esquema
 


