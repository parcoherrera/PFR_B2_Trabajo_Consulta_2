# Trabajo Consulta 2: Conexión base de datos relacional

## 1. ¿Qué es JDBC y cuáles son sus componentes?

JDBC (Java Database Connectivity) es una API de Java que permite conectar aplicaciones a bases de datos relacionales para realizar operaciones como consultas, inserciones, actualizaciones y eliminaciones. Actúa como un puente entre Java y las bases de datos, facilitando el trabajo con datos sin preocuparse por los detalles técnicos de la comunicación. JDBC utiliza un modelo de controladores (drivers) que traduce las llamadas de la API en comandos específicos para cada base de datos, y proporciona clases e interfaces para establecer conexiones, ejecutar sentencias SQL y procesar los resultados de manera estructurada.

#### Componentes:

##### 1. DriverManager
DriverManager es el encargado de gestionar los controladores JDBC y establecer conexiones entre la aplicación Java y la base de datos. Selecciona el controlador adecuado en función de la URL de conexión y proporciona métodos para registrar y desregistrar drivers.

##### 2. Connection
Connection representa una conexión activa con la base de datos, siendo la base para ejecutar consultas y gestionar transacciones. A través de este objeto, se pueden crear declaraciones SQL y configurar opciones como la gestión automática de transacciones.

##### 3. Statement
Statement permite ejecutar sentencias SQL estáticas, como consultas y actualizaciones, directamente sobre la base de datos. Es útil para sentencias simples, aunque no tan eficiente como PreparedStatement para operaciones repetitivas o parametrizadas.

##### 4. PreparedStatement
PreparedStatement es una versión mejorada de Statement que permite ejecutar consultas SQL parametrizadas. Ofrece mayor seguridad al evitar inyecciones SQL y es más eficiente al reutilizar sentencias compiladas en operaciones repetitivas con diferentes valores.

##### 5. CallableStatement
CallableStatement permite ejecutar procedimientos almacenados en la base de datos, facilitando operaciones complejas predefinidas. Admite parámetros de entrada y salida, permitiendo enviar datos a la base de datos y recuperar resultados generados por el procedimiento.

##### 6. ResultSet
ResultSet es el contenedor de resultados obtenidos tras ejecutar una consulta SQL. Proporciona métodos para iterar por filas de datos y acceder a las columnas mediante índices o nombres, facilitando la manipulación de los datos obtenidos.

##### 7. Driver
Driver es la interfaz que deben implementar los controladores específicos de cada base de datos. Su función es traducir las solicitudes JDBC en comandos comprensibles por el sistema de gestión de la base de datos correspondiente.

##### 8. SQLException
SQLException maneja errores relacionados con operaciones de bases de datos y conexiones en JDBC. Ofrece métodos para obtener mensajes de error, códigos específicos y estados SQL, permitiendo identificar y solucionar problemas durante la ejecución.

##### 9. DataSource
DataSource proporciona una forma avanzada y más eficiente de gestionar conexiones mediante un pool. Es utilizado en aplicaciones empresariales para optimizar el uso de recursos y mejorar el rendimiento en entornos con muchas solicitudes simultáneas.

## 2. Documente 2 librerías de Scala que permitan conectarse a una base de datos relacional. En una tabla resuma sus diferencias.

#### 1. Slick
Slick es una biblioteca de Scala que combina la programación funcional con el acceso a bases de datos relacionales. Ofrece consultas SQL tipadas en tiempo de compilación, evitando errores comunes y permitiendo un mapeo objeto-relacional (ORM) flexible. Su diseño modular soporta múltiples bases de datos como PostgreSQL, MySQL y SQLite. Además, facilita la programación reactiva y asincrónica, ideal para aplicaciones modernas. Con Slick, puedes trabajar con consultas SQL o abstraerte de ellas mediante su API declarativa.

#### 2. Doobie
Doobie es una biblioteca funcional pura para interactuar con bases de datos relacionales en Scala, construida sobre Cats. Ofrece un enfoque seguro para gestionar conexiones y transacciones mediante el uso de efectos (cats.effect). Sus consultas SQL tipadas en tiempo de compilación proporcionan una capa adicional de seguridad. Compatible con múltiples bases de datos, Doobie permite un control explícito sobre las consultas, haciendo que sea una opción ideal para desarrolladores que prefieren programación funcional y un manejo directo de SQL.

#### Tabla comparativa

| **Característica**              | **Slick**                                          | **Doobie**                                         |
|----------------------------------|----------------------------------------------------|---------------------------------------------------|
| **Enfoque**                      | Declarativo, más cercano a un ORM (Mapeo Objeto-Relacional) | Funcional puro, basado en Cats                    |
| **Interfaz**                     | Abstracción de consultas SQL con una API tipada    | Interpolación de SQL con un enfoque explícito y controlado |
| **Manejo de Conexiones**         | Abstracción automática de conexiones, con soporte para gestión reactiva | Manejo explícito de conexiones y transacciones usando `cats.effect.Resource` |
| **Consultas**                    | Consultas SQL generadas mediante clases Scala y tablas definidas | Consultas SQL declaradas como interpolaciones, con control manual |
| **Soporte de Bases de Datos**    | Múltiples bases de datos (PostgreSQL, MySQL, SQLite, etc.) | Múltiples bases de datos (PostgreSQL, H2, etc.) |
| **Manejo de Transacciones**      | Soporte para transacciones, pero con menos control explícito | Control total sobre transacciones con `Transactor` y `Resource` |
| **Compatibilidad con Programación Reactiva** | Soporte nativo con `Future` y `Akka Streams` | Compatible con `cats.effect.IO`, pero no tiene soporte nativo para streams reactivos |
| **Facilidad de Uso**             | Más accesible para quienes prefieren trabajar con un ORM y evitar SQL puro | Requiere mayor familiaridad con la programación funcional y efectos |
| **Seguridad en SQL**             | Previene inyecciones SQL con consultas tipadas | Previene inyecciones SQL mediante interpolación SQL tipada |
| **Popularidad**                  | Más popular entre los desarrolladores de Scala, especialmente en aplicaciones comerciales | Popular en comunidades que prefieren un enfoque funcional puro |

## 3. Documentar cómo establecer una conexión a una base de datos relacional (mysql). Siga los siguientes pasos:

#### Generar una base de datos con una tabla con datos de prueba

<img width="711" alt="image" src="https://github.com/user-attachments/assets/c2d1e24b-4248-497c-be78-b0748f7afd20" />

<img width="354" alt="image" src="https://github.com/user-attachments/assets/67a211b7-3348-4155-b516-b196cfeafae5" />

#### Desde Scala establecer la conexión a la Base Datos.

```scala
import slick.jdbc.MySQLProfile.api._
import scala.concurrent.Await
import scala.concurrent.duration._
import slick.lifted.{ProvenShape, Tag}

case class Song(trackNumber: Int, songTitle: String, albumName: String, duration: String)

class Songs(tag: Tag) extends Table[Song](tag, "queen_album_songs") {
  def trackNumber: Rep[Int] = column[Int]("track_number", O.PrimaryKey)
  def songTitle: Rep[String] = column[String]("song_title")
  def albumName: Rep[String] = column[String]("album_name")
  def duration: Rep[String] = column[String]("duration")

  def * : ProvenShape[Song] = (trackNumber, songTitle, albumName, duration) <> (
    (t: (Int, String, String, String)) => Song(t._1, t._2, t._3, t._4),
    (s: Song) => (s.trackNumber, s.songTitle, s.albumName, s.duration)
  )
}

object Main extends App {
  // Crear el objeto de base de datos
  val db = Database.forURL("jdbc:mysql://localhost:3306/my_db", driver = "com.mysql.cj.jdbc.Driver", user = "root", password = "79513marco")

  // Definir la tabla de canciones
  val songs = TableQuery[Songs]

  // Consultar las canciones de la tabla
  val query = songs.result

  // Ejecutar la consulta
  val future = db.run(query)

  // Esperar y mostrar el resultado
  val result = Await.result(future, 10.seconds)
  result.foreach { song =>
    println(s"Track ${song.trackNumber}: ${song.songTitle} (${song.duration})")
  }

  // Cerrar la conexión
  db.close()
}
```

#### Consulta de todos los datos de la tabla de prueba.

<img width="420" alt="image" src="https://github.com/user-attachments/assets/c5ee3ed4-6047-416f-ba37-d1715a67ca7c" />

