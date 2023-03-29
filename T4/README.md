# Generación de tipos

A la hora de definir los tipos, estos pueden ser:

- Enumarados 

```sql
create type perfil as enum ('estudiante', 'docente','staff')
```

- Estructurados

```sql
create type matricula as ( numero integer, nia integer )
```

Estos tipos de datos se pueden utilizar a la hora de crear elementos en la base de datos.

```sql
insert into alumnos (id,nombre,apellido,matricula,perfil) values (1,'Borja','Martin',(1,1),'estudiante')
```

# Generación de clases

Las clases en postgres representan tablas. Estas tablas son las que luego en programación será mostradas

```sql
create table aulas (id serial primary key, nombre varchar, puestos integer, pizarra boolean, dispositivos varchar[])
```

En el caso de querer agregar un dato a dicha tabla utilizaríamos la siguiente sintaxis

```sql
insert into aulas (nombre, puestos, pizarra, dispositivos) values ('paris',20,true, '{"proyector", "hub", "television"}')
```

Como se puede ver además de los datos típicos, se pueden guardar arrays. Es importante saver que este array no es solo de los tipos de datos básicos, sino que también puede ser de tipos de datos ya creados. 

```sql
create table aulas (id serial primary key, nombre varchar, puestos integer, pizarra boolean, dispositivos dispositivo[])
insert into aulas (nombre, puestos, pizarra, dispositivos) values ('paris',20,true, array[(1,'pizarra','pizarra electronica'),(2,'television','television para videollamadas')]::dispositivo[])
```

## Herencia

Al igual que en programación las clases - tablas también pueden heredar. Para ello se utiliza la palabra reservada inherits

Supongamos la tabla que hemos trabajado antes de aula

```sql
create table aulas (id serial primary key, nombre varchar, puestos integer, pizarra boolean, dispositivos varchar[])
```

Si queremos hacer una tabla especializada para guardar aulas multimedia y aulas teoria, podríamos tener dos tablas adicionales, cada una de ellas extendiendo de la tabla aulas

``` sql
create table aula_multimedia (os varchar, específica boolean) inherits (aulas)
create table aula_teoria (mesas int, biblioteca boolean) inherits (aulas)
```

En el caso de querer agregar cosas a las tablas, tendríamos que tener en cuenta todos los datos que provienen de la tabla aulas ya que han sido heredados

``` sql
insert into aula_multimedia (nombre, puestos, pizarra, os, especifica) values ('atenas',18,true,'windows',true)
insert into aula_teoria (nombre, puestos, pizarra, mesas, biblioteca) values ('berlin',25,true,30,false)
```

En este caso, si hacemos un select de todos los elementos partiendo del general, aparecerán todos los dato

``` sql
select * from aulas
```

En el caso de querer solo aquellas aulas que pertenezcan a la clase general y no a todas se utiliza la palabra ONLY y se obian las clases que han hererado

```sql
select * from only aulas
```

Si alguno de las columnas del objeto de tipo array, podemos mostrar todos los elementos de forma individual utilizando el comando unnest

```sql
select id, nombre,unnest(dispositivos) from only aulas
```

# Conexión con programación

A la hora de conectar una base de datos Postgres con un lenguaje de programación como por ejemplo java, se realiza mediante el JDBC

```java
public class Conexion {

    private Connection connection = null;

    private void connect(){
        try{
            String url = "jdbc:postgresql://localhost/docencia";
            connection = DriverManager.getConnection(url, "postgres","");
        }catch (Exception e){
            System.out.println("Fallo en la conexion");
        }
    }

    public Connection getConnection(){
        if (connection==null){
            connect();
        }
        return  connection;
    }

    public void disconnect(){
        if (connection!=null){
            try {
                connection.close();
            } catch (SQLException e) {
                throw new RuntimeException(e);
            }
        }
    }
}

```

Y para poder ejecutar métodos se utilizan los objetos statement, preparestatement y resultset

```java
public class Operaciones {

    Connection connection;
    Conexion conexion;

    public Operaciones(){
        conexion = new Conexion();
        connection = conexion.getConnection();
        try {
            System.out.printf(connection.getSchema());
        } catch (SQLException e) {
            throw new RuntimeException(e);
        }
    }

    public void agregarDatos(){
        try {
            Statement st = connection.createStatement();
            st.executeUpdate("INSERT into aulas (nombre, puestos, pizarra) VALUES ('londres',15,false)");
            st.close();

        } catch (SQLException e) {
            throw new RuntimeException(e);
        }
    }

    public void consultarDatos(){
        try {
            Statement statement =  connection.createStatement();
            ResultSet resultSet = statement.executeQuery("SELECT * from aulas");
            while (resultSet.next()){
                String nombre = resultSet.getString("nombre");
                int puestos  = resultSet.getInt("puestos");
                boolean pizarra = resultSet.getBoolean("pizarra");
                System.out.printf("Nombre: %s Puestos: %d Pizzada %b%n",nombre,puestos,pizarra);
            }

        } catch (SQLException e) {
            throw new RuntimeException(e);
        }
    }

}

```