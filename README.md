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
La persistencia en Hibernate se refiere al proceso de guardar, recuperar, actualizar y eliminar datos de la base de datos utilizando tus objetos Java. La optimización busca asegurar que estas operaciones se realicen de la manera más eficiente posible, minimizando el uso de recursos y maximizando la velocidad de la aplicación.

<a id="operaciones-crud"></a>
### Operaciones CRUD
CRUD es un acrónimo que representa las cuatro operaciones básicas de persistencia:

1, Create (Crear): Guardar un nuevo objeto en la base de datos. En Hibernate, esto se logra principalmente utilizando los métodos save(), persist(), y merge() de la interfaz Session.

2. Read (Leer): Recuperar un objeto existente de la base de datos. Los métodos principales para esto son get(), load(), y las consultas a través de HQL, Criteria API o SQL nativo.

3. Update (Actualizar): Modificar el estado de un objeto persistente y sincronizar estos cambios con la base de datos. Esto se realiza principalmente a través del método update() o simplemente modificando un objeto que ya está en estado persistente dentro de una transacción (auto-dirty checking). También se puede usar merge() para actualizar un objeto desvinculado.

4. Delete (Borrar): Eliminar un objeto de la base de datos. Se utiliza el método delete() de la interfaz Session.

<a id="uso-de-session-en-crud"></a>
#### Uso del Session para operaciones básicas

> Operación create
```
Session session = sessionFactory.openSession();
Transaction transaction = null;
try {
    transaction = session.beginTransaction();

    Usuario nuevoUsuario = new Usuario();
    nuevoUsuario.setNombre("Juan Pérez");
    nuevoUsuario.setEmail("juan.perez@example.com");

    // save(): Guarda el objeto y le asigna un identificador inmediatamente.
    Long idGuardado = (Long) session.save(nuevoUsuario);
    System.out.println("ID del usuario guardado: " + idGuardado);

    // persist(): Guarda el objeto, pero no garantiza la asignación inmediata del identificador.
    // Es preferible para la semántica de JPA.
    Pedido nuevoPedido = new Pedido();
    nuevoPedido.setFechaPedido(new java.util.Date());
    session.persist(nuevoPedido);

    transaction.commit();
} catch (Exception e) {
    if (transaction != null) {
        transaction.rollback();
    }
    e.printStackTrace();
} finally {
    session.close();
}
```

> Operación read
```
Session session = sessionFactory.openSession();
try {
    // get(): Recupera el objeto inmediatamente. Si no existe, devuelve null.
    Usuario usuario1 = session.get(Usuario.class, 1L);
    if (usuario1 != null) {
        System.out.println("Usuario encontrado (get): " + usuario1.getNombre());
    } else {
        System.out.println("Usuario con ID 1 no encontrado (get).");
    }

    // load(): Recupera un proxy del objeto inmediatamente. El objeto real se carga solo cuando se accede a sus propiedades.
    // Si el objeto no existe en la base de datos, lanza una excepción (ObjectNotFoundException) al acceder a sus propiedades.
    Usuario usuario2 = session.load(Usuario.class, 2L);
    System.out.println("Proxy de usuario cargado (load).");
    // Intentar acceder a una propiedad forzará la carga del objeto.
    try {
        System.out.println("Nombre del usuario (load): " + usuario2.getNombre());
    } catch (org.hibernate.ObjectNotFoundException e) {
        System.out.println("Usuario con ID 2 no encontrado (load).");
    }

    // Consultas (HQL, Criteria, Native SQL) se utilizan para leer conjuntos de objetos o realizar búsquedas más complejas.
    // Ejemplo con HQL:
    java.util.List<Usuario> usuarios = session.createQuery("from Usuario where nombre like :nombre", Usuario.class)
            .setParameter("nombre", "J%").list();
    for (Usuario u : usuarios) {
        System.out.println("Usuario encontrado por nombre: " + u.getNombre());
    }

} catch (Exception e) {
    e.printStackTrace();
} finally {
    session.close();
}
```

> Operación update
```
Session session = sessionFactory.openSession();
Transaction transaction = null;
try {
    transaction = session.beginTransaction();

    // Método 1: Cargar el objeto y modificarlo (auto-dirty checking).
    Usuario usuarioParaActualizar = session.get(Usuario.class, 1L);
    if (usuarioParaActualizar != null) {
        usuarioParaActualizar.setEmail("juan.perez.actualizado@example.com");
        // No es necesario llamar a un método update() explícitamente.
        // Hibernate detectará los cambios al final de la transacción (o al hacer flush).
    }

    // Método 2: Usar el método update() para un objeto que ya está asociado a la sesión.
    if (usuarioParaActualizar != null) {
        session.update(usuarioParaActualizar);
    }

    // Método 3: Usar merge() para actualizar un objeto desvinculado.
    Usuario usuarioDesvinculado = new Usuario();
    usuarioDesvinculado.setId(2L);
    usuarioDesvinculado.setNombre("Carlos López (Actualizado)");
    Usuario usuarioMergeado = (Usuario) session.merge(usuarioDesvinculado);
    System.out.println("Usuario mergeado con ID: " + usuarioMergeado.getId());

    transaction.commit();
} catch (Exception e) {
    if (transaction != null) {
        transaction.rollback();
    }
    e.printStackTrace();
} finally {
    session.close();
}
```

> Operación delete
```
Session session = sessionFactory.openSession();
Transaction transaction = null;
try {
    transaction = session.beginTransaction();

    Usuario usuarioParaBorrar = session.get(Usuario.class, 3L);
    if (usuarioParaBorrar != null) {
        session.delete(usuarioParaBorrar);
        System.out.println("Usuario con ID 3 borrado.");
    } else {
        System.out.println("Usuario con ID 3 no encontrado para borrar.");
    }

    transaction.commit();
} catch (Exception e) {
    if (transaction != null) {
        transaction.rollback();
    }
    e.printStackTrace();
} finally {
    session.close();
}
```

> [!IMPORTANT]
> Siempre se deben de manejar las transacciones correctamente (iniciar, confirmar o deshacer) y cerrar la Session en un bloque finally para liberar los recursos.

<a id="cache-de-hibernate"></a>
### Caché de Hibernate
El caché de Hibernate es un mecanismo crucial para mejorar el rendimiento de las aplicaciones al reducir la frecuencia con la que se accede a la base de datos. Hibernate ofrece varios niveles de caché.

<a id="cache-de-primer-nivel"></a>
#### Caché de primer nivel
> Características:

1. El caché de primer nivel está asociado a una instancia de Session.
2. Es habilitado por defecto y **no se puede deshabilitar**.
3. Cuando Hibernate carga un objeto desde la base de datos dentro de una sesión, lo almacena en el caché de primer nivel por su identificador. Si se solicita el mismo objeto nuevamente dentro de la misma sesión, Hibernate lo recupera del caché en lugar de ir a la base de datos.
4. Los cambios realizados en un objeto persistente dentro de la sesión también se reflejan en el caché de primer nivel.
5. **El caché de primer nivel no se comparte entre diferentes sesiones**. Cada nueva sesión tiene su propio caché.
6. La vida útil de los objetos en el caché de primer nivel está limitada a la vida útil de la Session. **Cuando la sesión se cierra, el caché se descarta**.

> [!IMPORTANT]
> Debido a que está asociado a la sesión, su alcance es limitado a una única transacción o unidad de trabajo.

<a id="cache-de-segundo-nivel"></a>
#### Caché de segundo nivel
El caché de segundo nivel está asociado a la SessionFactory y **es compartido por todas las sesiones creadas por esa fábrica**. Su objetivo es almacenar objetos y datos de consulta que pueden ser reutilizados entre diferentes sesiones y transacciones, mejorando significativamente el rendimiento

- Hibernate permite la integración con varios proveedores de caché de segundo nivel, como:
1. Ehcache: Un proveedor de caché distribuido en memoria.
2. Hazelcast: Una plataforma de computación en memoria distribuida.
3. Infinispan: Un data grid distribuido y un caché en memoria.
4. Redis: Un almacén de estructuras de datos en memoria.
5. Memcached: Un sistema de caché distribuido en memoria.

> Configuración del Caché de Segundo Nivel

1. Agregar la dependencia del proveedor de caché dentro del proyecto (por ejemplo, la dependencia de Ehcache en Maven o Gradle).

2. Configurar las propiedades del caché de segundo nivel en el archivo hibernate.cfg.xml o mediante configuración programática. Esto incluye especificar el proveedor de caché y cualquier configuración específica del proveedor.
```
<hibernate-configuration>
    <session-factory>
        <property name="hibernate.cache.use_second_level_cache">true</property>
        <property name="hibernate.cache.region.factory_class">org.hibernate.cache.ehcache.EhCacheRegionFactory</property>
        </session-factory>
</hibernate-configuration>
```

3. Marcar las entidades que deben ser cacheadas utilizando la anotación @Cache (de JPA o Hibernate). Se puede especificar la estrategia de concurrencia del caché.
```
import jakarta.persistence.Entity;
import jakarta.persistence.Id;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.GenerationType;
import org.hibernate.annotations.Cache;
import org.hibernate.annotations.CacheConcurrencyStrategy;

@Entity
@Cache(usage = CacheConcurrencyStrategy.READ_WRITE)
public class Producto {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String nombre;
    private java.math.BigDecimal precio;
    // ...
}
```

- Estrategias de Concurrencia del Caché de Segundo Nivel (CacheConcurrencyStrategy):

* `READ_ONLY`: Para entidades que nunca cambian. Es la estrategia más segura y rápida.
* `NONSTRICT_READ_WRITE`: Para entidades que cambian con poca frecuencia y la consistencia estricta no es crítica.
* `READ_WRITE`: Para entidades que pueden cambiar, Hibernate utiliza un mecanismo de bloqueo suave para garantizar la consistencia.
* `TRANSACTIONAL`: Para entornos JTA donde el caché se sincroniza con las transacciones JTA.

> [!NOTE]
> El caché de segundo nivel se organiza en regiones. Por defecto, Hibernate utiliza una región por entidad cacheada. También se pueden cachear los resultados de las consultas (query cache)


<a id="otros-niveles-de-caches"></a>
#### Otros niveles de caché

* `Caché de Nivel Web (HTTP Cache)`: Para aplicaciones web, el uso de encabezados de caché HTTP (como Cache-Control, Expires, ETag, Last-Modified) puede reducir la cantidad de solicitudes al servidor al almacenar respuestas en el navegador o en servidores proxy.

* `Caché de Base de Datos`: Muchas bases de datos tienen sus propios mecanismos de caché para optimizar las consultas. Comprender y configurar el caché de la base de datos también es importante para el rendimiento general.


<a id="estrategias-de-carga"></a>
### Fetch Strategies (Estrategias de Carga)
Las estrategias de carga **definen cómo Hibernate recupera las entidades relacionadas (asociaciones)**. La elección de la estrategia de carga puede tener un impacto significativo en el número de consultas a la base de datos y, por lo tanto, en el rendimiento de la aplicación. Las dos estrategias principales son:

> Eager Loading (Carga Ansiosa)
Las entidades relacionadas **se cargan al mismo tiempo que la entidad principal**, en la misma consulta SQL (generalmente utilizando JOINs).

Se configura utilizando el atributo fetch = FetchType.EAGER en las anotaciones de las relaciones (@OneToOne, @ManyToOne, @OneToMany, @ManyToMany).

- Ventajas
1. Evita el problema del "N+1 selects".

- Desventajas
1. Puede cargar una gran cantidad de datos innecesarios si no siempre se necesitan las entidades relacionadas, lo que puede consumir memoria

> Lazy Loading (Carga Perezosa)

Las entidades relacionadas no se cargan inmediatamente. Se carga un "proxy" (un objeto marcador de posición). Los datos reales de la entidad relacionada **se cargan solo cuando se accede a sus propiedades por primera vez** (lo que generalmente resulta en una consulta adicional a la base de datos).

Es la estrategia de carga por defecto para las colecciones (@OneToMany, @ManyToMany) y puede configurarse para relaciones individuales (@OneToOne, @ManyToOne) utilizando fetch = FetchType.LAZY

- Ventajas
1. Carga solo los datos necesarios cuando se solicitan, lo que puede mejorar el rendimiento inicial y reducir el consumo de memoria

- Desventajas
1. Se puede llevar al problema del "N+1 selects". Si cargas N entidades y cada una tiene una colección relacionada que se carga de forma perezosa al acceder a ella, Hibernate podría ejecutar N+1 consultas a la base de datos (una para cargar las N entidades y N consultas adicionales para cargar las colecciones de cada entidad).
2. Requiere que la Session esté abierta cuando se accede a las relaciones cargadas perezosamente.

Para mitigar el problema del "N+1 selects" con la carga perezosa, Hibernate ofrece varias estrategias de fetching que se pueden especificar en las consultas (HQL o Criteria API):

- `JOIN FETCH (o FETCH JOIN en HQL)`: Fuerza la carga ansiosa de la asociación especificada en la consulta, anulando la estrategia de carga definida en el mapeo de la entidad.
```
SELECT c FROM Cliente c JOIN FETCH c.pedidos WHERE c.id = :clienteId
```

- `Batch Fetching`: Permite a Hibernate cargar múltiples colecciones perezosas en una sola consulta cuando se accede a la primera de ellas. Se configura mediante las anotaciones @BatchSize (en la colección) o @Fetch(FetchMode.SUBSELECT) (para cargar todas las colecciones de las entidades cargadas en una subconsulta).

- `Subselect Fetching (@Fetch(FetchMode.SUBSELECT))`: Carga todas las colecciones perezosas de las entidades padre cargadas en una sola subconsulta. Esto puede ser útil cuando se cargan múltiples entidades padre y se necesita acceder a sus colecciones.

  
<a id="consultas-y-recuperacion-de-datos"></a>
## Consultas y recuperación de datos
Hibernate ofrece varias formas de consultar y recuperar datos de la base de datos, cada una con sus propias características y casos de uso. Las principales son HQL, Criteria API y Native SQL.

<a id="HQL"></a>
### HQL (Hibernate Query Language)
HQL es un `lenguaje de consulta orientado a objetos que es sensible a mayúsculas y minúsculas` (excepto para las palabras clave HQL). Se parece a SQL, pero en lugar de operar sobre tablas y columnas, opera sobre entidades y sus atributos. Hibernate **traduce las consultas HQL al SQL específico del dialecto de la base de datos subyacente**.

> Principales características
1. Opera sobre entidades y sus propiedades: En lugar de SELECT * FROM tabla, en HQL escribirías algo como FROM Entidad.
2. Soporta asociaciones: Se puede navegar por las relaciones entre entidades directamente en las consultas (usando la notación de punto).
3. Polimorfismo: Las consultas sobre una superclase también pueden devolver instancias de sus subclases (dependiendo de la estrategia de herencia).
4. Parámetros con nombre y posicionales: Permite escribir consultas más seguras y legibles.
5. Funciones y operadores: Soporta muchas de las funciones y operadores SQL estándar, así como algunas específicas de Hibernate.
6. Paginación y ordenamiento: Facilita la recuperación de datos en lotes y la ordenación de los resultados.
7. Proyecciones y agregaciones: Permite seleccionar atributos específicos y realizar funciones de agregación.

> Ejemplos
- Seleccionar todas las entidades de un tipo
```
List<Usuario> usuarios = session.createQuery("FROM Usuario", Usuario.class).list();
```

- Seleccionar entidades con una condición (cláusula WHERE)
```
List<Pedido> pedidosDeCliente = session.createQuery("FROM Pedido WHERE cliente.id = :clienteId", Pedido.class)
        .setParameter("clienteId", clienteId).list();
```

- Realizar joins explícitos
```
List<Object[]> clientesConPedidos = session.createQuery("SELECT c, p FROM Cliente c JOIN c.pedidos p").list();
```

- Usar funciones de agregación
```
Long totalPedidos = session.createQuery("SELECT COUNT(*) FROM Pedido", Long.class).uniqueResult();
Double promedioPrecio = session.createQuery("SELECT AVG(p.precio) FROM Producto p", Double.class).uniqueResult();
```

Las consultas HQL se ejecutan utilizando la interfaz Query obtenida de la Session.
```
Query<Usuario> query = session.createQuery("FROM Usuario WHERE edad > :edad", Usuario.class);
query.setParameter("edad", 30);
List<Usuario> usuariosMayoresDe30 = query.list();
```

<a id="criteria-api"></a>
### Criteria API

La Criteria API es una API programática para construir consultas Hibernate de forma dinámica utilizando objetos Java en lugar de cadenas de texto HQL.

> Componentes Principales de la Criteria API:

- CriteriaBuilder: Una fábrica para crear objetos de consulta (CriteriaQuery), expresiones, predicados (restricciones) y funciones de agregación. Se obtiene de la EntityManagerFactory (en un contexto JPA) o de la Session (en Hibernate puro).

- CriteriaQuery: Representa una consulta completa. Define la entidad raíz de la consulta (la cláusula FROM) y puede especificar el tipo del resultado (por ejemplo, una entidad o un tipo específico).

- Root<T>: Representa la entidad raíz de la consulta (la tabla principal en SQL). Permite acceder a los atributos de la entidad para definir restricciones y selecciones.

- Predicate: Representa una condición o restricción (la cláusula WHERE en SQL). Se construyen utilizando métodos del CriteriaBuilder (por ejemplo, equal(), like(), greaterThan()).

- Selection: Define lo que se va a seleccionar (la cláusula SELECT en SQL). Puede ser la entidad completa o atributos específicos.

- Order: Define el criterio de ordenamiento (la cláusula ORDER BY en SQL).

> Ejemplos

- Seleccionar entidades con una condición
```
  CriteriaBuilder cb = session.getCriteriaBuilder();
  CriteriaQuery<Pedido> cq = cb.createQuery(Pedido.class);
  Root<Pedido> root = cq.from(Pedido.class);
  Predicate estadoPendiente = cb.equal(root.get("estado"), "PENDIENTE");
  cq.where(estadoPendiente);
  List<Pedido> pedidosPendientes = session.createQuery(cq).list();
```

- Seleccionar atributos específicos (proyección)
```
  CriteriaBuilder cb = session.getCriteriaBuilder();
  CriteriaQuery<String> cq = cb.createQuery(String.class);
  Root<Usuario> root = cq.from(Usuario.class);
  cq.select(root.get("nombre"));
  List<String> nombresDeUsuario = session.createQuery(cq).list();
  
  CriteriaQuery<Object[]> cq2 = cb.createQuery(Object[].class);
  Root<Usuario> root2 = cq2.from(Usuario.class);
  cq2.multiselect(root2.get("nombre"), root2.get("email"));
  List<Object[]> nombreYEmail = session.createQuery(cq2).list();
```

- Usar parámetros
```
  CriteriaBuilder cb = session.getCriteriaBuilder();
  CriteriaQuery<Pedido> cq = cb.createQuery(Pedido.class);
  Root<Pedido> root = cq.from(Pedido.class);
  Join<Pedido, Cliente> clienteJoin = root.join("cliente");
  Predicate clienteEspecifico = cb.equal(clienteJoin.get("id"), clienteId);
  cq.where(clienteEspecifico);
  List<Pedido> pedidosDeCliente = session.createQuery(cq)
          .setParameter("clienteId", clienteId) // No se define aquí, se pasa al crear la Query
          .list();
```

> [!NOTE]
> 
> La Criteria API es especialmente útil para construir consultas dinámicas y complejas donde las condiciones varían en tiempo de ejecución.

<a id="native-sql"></a>
### Native SQL
Hibernate también permite ejecutar consultas SQL nativas directamente contra la base de datos. Esto puede ser útil cuando necesitas utilizar características específicas de la base de datos que no están soportadas por HQL o la Criteria API.

Para la ejecución de consultas nativas se utiliza el método createNativeQuery() de la Session.
> Ejemplo
```
// Mapear el resultado a una entidad
List<Usuario> usuariosPorSQL = session.createNativeQuery("SELECT * FROM usuarios WHERE edad > :edad", Usuario.class)
        .setParameter("edad", 25)
        .list();

// Obtener un resultado escalar
Object nombreUsuario = session.createNativeQuery("SELECT nombre FROM usuarios WHERE id = :id")
        .setParameter("id", 1L)
        .uniqueResult();

// Obtener una lista de arrays de objetos
List<Object[]> resultados = session.createNativeQuery("SELECT nombre, email FROM usuarios WHERE activo = 1")
        .list();
for (Object[] resultado : resultados) {
    String nombre = (String) resultado[0];
    String email = (String) resultado[1];
    System.out.println("Nombre: " + nombre + ", Email: " + email);
}

// Usar ResultTransformer para mapear a un DTO (Data Transfer Object)
List<UsuarioDTO> usuariosDTO = session.createNativeQuery("SELECT id, nombre, email FROM usuarios WHERE activo = 1")
        .setResultTransformer(Transformers.aliasToBean(UsuarioDTO.class))
        .list();
```

- Consideraciones
1. Las consultas SQL nativas son específicas de la base de datos subyacente. Cambiar la base de datos requerirá modificar estas consultas.
2. No hay verificación de sintaxis por parte de Hibernate hasta el momento de la ejecución.
   
<a id="proyecciones-y-agregaciones"></a>
### Proyecciones y agregaciones
Tanto HQL como la Criteria API permiten seleccionar proyecciones (subconjuntos de los atributos de una entidad) y realizar funciones de agregación sobre los datos.

- Proyecciones

Las proyecciones permiten **seleccionar atributos específicos de una entidad** en lugar de la entidad completa. Esto puede ser útil para optimizar las consultas cuando solo necesitas ciertos datos.

- Agregaciones

Hibernate soporta las funciones de agregación estándar de SQL como COUNT(), AVG(), SUM(), MIN(), MAX(). Estas funciones se pueden utilizar tanto en HQL como en la Criteria API

> Ejemplo de proyección y agregación combinados (HQL)
```
List<Object[]> resumenPedidos = session.createQuery(
        "SELECT c.nombre, COUNT(p), AVG(p.total) " +
        "FROM Cliente c JOIN c.pedidos p " +
        "GROUP BY c.nombre " +
        "ORDER BY COUNT(p) DESC")
        .list();

for (Object[] resultado : resumenPedidos) {
    String nombreCliente = (String) resultado[0];
    Long cantidadPedidos = (Long) resultado[1];
    Double promedioTotal = (Double) resultado[2];
    System.out.println("Cliente: " + nombreCliente + ", Pedidos: " + cantidadPedidos + ", Promedio Total: " + promedioTotal);
}
```

> Ejemplo de proyección y agregación combinados (Criteria API):
```
CriteriaBuilder cb = session.getCriteriaBuilder();
CriteriaQuery<Object[]> cq = cb.createQuery(Object[].class);
Root<Cliente> clienteRoot = cq.from(Cliente.class);
Join<Cliente, Pedido> pedidoJoin = clienteRoot.join("pedidos");

cq.multiselect(clienteRoot.get("nombre"), cb.count(pedidoJoin), cb.avg(pedidoJoin.get("total")))
        .groupBy(clienteRoot.get("nombre"))
        .orderBy(cb.desc(cb.count(pedidoJoin)));

List<Object[]> resumenPedidos = session.createQuery(cq).list();

for (Object[] resultado : resumenPedidos) {
    String nombreCliente = (String) resultado[0];
    Long cantidadPedidos = (Long) resultado[1];
    Double promedioTotal = (Double) resultado[2];
    System.out.println("Cliente: " + nombreCliente + ", Pedidos: " + cantidadPedidos + ", Promedio Total: " + promedioTotal);
}
```

<a id="temas-avanzados"></a>
## Temas avanzados

<a id="interceptores-y-listeners"></a>
### Interceptores y listeners

<a id="generacion-de-esquema"></a>
### Generación de esquema
 


