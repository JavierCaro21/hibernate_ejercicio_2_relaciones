# Hibernate - Ejercicio 2 - Relaciones

## 1 - Duplicamos el ejercicio 1 y cremos una nueva BD
Nuevo proyecto y nueva base de datos

 - Duplicamos el ejercicio 1. Eliminamos los ficheros que no nos hagan falta
   - Cremos un nuevo repositorio en gitHub y lo subimos
   - Creamos una nueva BD `relaciones_hibernate` con las tablas `detalles_cliente` y `cliente` que tendrán una relación `1:1`

   ```sql
      create table detalles_cliente
      (
          id int auto_increment,
          web varchar(128) null,
          telefono varchar(50) null,
          comentarios varchar(128) null,
          constraint detalles_cliente_pk primary key (id)
      );

      create table cliente
      (
         id int auto_increment,
         nombre varchar(128) null,
         apellido varchar(128) null,
         direccion varchar(128) null,
         constraint cliente_pk primary key (id),
         constraint cliente_detalles_cliente_id_fk foreign key (id) references detalles_cliente (id)
      );
   ```

 - Adaptamos los ficheros de configuración a nuestra nueva base de datos
 - Comprobamos que la conexión funciona correctamente

## 2 - Clase cliente y DetallesCliente
Ahora crearemos las clases cliente y detalles cliente

 - Creamos una clase con el mismo nombre que la tabla detalles_cliente `DetallesCliente.java`
   - Creamos los campos acorde a la tabla
   - Añadimos las anotaciones de Hibernate `@Entity`, `@Table`, `@Id`, `@Column` (si hace falta), constructor, toString y los getter y setters

   ```java
        @Entity
        @Table(name="cliente")
        public class Cliente {

            @Id
            @GeneratedValue(strategy = GenerationType.IDENTITY)
            private int id;
            
            private String nombre;
            
            @Column(name = "apellido")
            private String apellidos;
            
            private String direccion;
            
            @OneToOne(cascade=CascadeType.ALL)
            @JoinColumn(name = "id")
            private DetallesCliente detallesCliente;
            
            @OneToMany(mappedBy = "cliente", cascade = {CascadeType.PERSIST, CascadeType.MERGE, CascadeType.DETACH, CascadeType.REFRESH})
            private List<Pedido> listaPedidos;
    
            //...
   ```

 - Creamos una clase con el mismo nombre que la tabla cliente `Cliente.java`
   - Creamos los campos acorde a la tabla
   - Añadimos las anotaciones de Hibernate `@Entity`, `@Table`, `@Id`, `@Column` (si hace falta), constructor, toString y los getter y setters

   ```java
    @Entity
    @Table(name="detalles_cliente")
    public class DetallesCliente {

        @Id
        @GeneratedValue(strategy = GenerationType.IDENTITY)
        private int id;
    
        private String web;
    
        private String telefono;
    
        private String comentarios;
    
        @OneToOne(mappedBy = "detallesCliente", cascade = CascadeType.ALL) //Esto consigue la bidireccionalidad en la 1:1. Elimina en cascada
        private Cliente elCliente;
   
        //...
    ```
## 3 - Leer datos de la base de datos

 - Añadimos que imprima el id y veremos que siempre es 0
 - Hay que añadir al id para que lo lea
   ```java
    @GeneratedValue(strategy = GenerationType.IDENTITY)
   ```
   - Ahora ya recupera el id que acabamos de insertar
 - Creamos una clase similar a `GuardaCliente.java` pero esta vez que lea también los detalles del cliente
   - `LeeDetallesCliente.java`
   ```java
    // Leemos el objeto de tipo cliente de nuestra base de datos
    DetallesCliente detallesCliente = miSession.get(DetallesCliente.class, 1);

    //Si ha ido bien nos mostrará el mensaje
    System.out.println("Cliente " +  detallesCliente.getElCliente().getNombre() + " obtenido de la base de datos a través de sus detalles!!");
    System.out.println(detallesCliente);
   ```
## 4 - Leer lista de clientes con hql

 - Creamos una nueva clase `LeerClientes.java` e introducimos una query hql para leer los clientes.
    ```java
      // consulta de clientes con HQL
      String hql = "from Cliente";
      List<Cliente> clientesLeidos = miSession.createQuery(hql).getResultList();
    ```
 - También podemos imprimir los de apellido Martinez
    ```java
      //Ahora consulta con los que su apellido sea igual a Martinez
      hql = "from Cliente c1 where c1.apellidos='Martinez'";
      clientesLeidos = miSession.createQuery(hql).getResultList();
      System.out.println("- Hay " + clientesLeidos.size() + " con ese apellido");
    ```

 - Creamos una nueva clase `LeeDetallesCliente.java` y leemos los detalles de cliente de id 1 y mostramos el nombre del cliente por pantalla.
    ```java
      // Leemos el objeto de tipo cliente de nuestra base de datos
      DetallesCliente detallesCliente = miSession.get(DetallesCliente.class, 1);

      //Si ha ido bien nos mostrará el mensaje
      System.out.println("Cliente " +  detallesCliente.getElCliente().getNombre() + " obtenido de la base de datos a través de sus detalles!!");
      System.out.println(detallesCliente);
    ```

## 5 - Eliminar varios de clientes con hql

- Creamos una nueva clase `EliminarClientes.java`, muy parecida a `LeerClientes.java` pero esta vez para eliminarl los que el nombre empiece por "Mar". Para ello introducimos una query hql para eliminar los clientes. Se eliminarán en cascada los clientes.

## 6 - Eliminar los detalles de un cliente

- Creamos una nueva clase `EliminarDetallesCliente.java`, muy parecida a `LeeDetallesCliente.java` pero esta vez para eliminar los detalles de cliente de id 4 y en consecuencia el cliente asociado (se eliminará automáticamente en cascada).

   ```java
      @OneToOne(mappedBy = "detallesCliente", cascade = CascadeType.ALL)
   ```

### ¿Cómo Eliminar los detalles de un cliente sin eliminar el cliente?
Este no lo vamos a hacer, pero qué pasa si no quiero que se elimine en cascada? Pues conforme hemos diseñado la base de datos no podemos, pero deberíamos de seguir los siguientes pasos:

   - Quitar de DetallesCliente lo de cascade
   
   ```java
      @OneToOne(mappedBy = "detallesCliente")
   ```

   - Obtener el cliente y setear a null sus detalles antes de hacer el delete.
   - Eliminar el FK de clientes que apunta a detalles cliente

## 7 - Guardar una lista de Pedidos en cliente. Relación 1:M. 
Vamos a crear una nueva tabla en la que se puedan relacionar los clientes con pedidos. Un cliente podrá tener muchos pedidos y si elimina un pedido no eliminaremos al cliente en cascada :)

   - Creamos la tabla pedido

   ```sql
      create table pedido
      (
         id int auto_increment,
         fecha DATE null,
         forma_pago varchar(30) null,
         cliente_id int null,
         constraint pedido_pk
         primary key (id),
         constraint pedido_cliente_id_fk
         foreign key (cliente_id) references cliente (id)
      );
   ```

   - El siguiente paso es crear la clase `Pedido.java` para esta tabla
     - Tendrá una referencia a cliente y esta vez no le pondremos todos los cascade

      ```java
         @ManyToOne(cascade = {CascadeType.PERSIST, CascadeType.MERGE, CascadeType.DETACH, CascadeType.REFRESH}) 
         @JoinColumn(name = "cliente_id")
         private Cliente cliente;
      ```

  - Modificaremos la clase Cliente y le añadiremos una lista de pedidos y un método para añadir un pedido
  
      ```java
         @OneToMany(mappedBy = "cliente", cascade = {CascadeType.PERSIST, CascadeType.MERGE, CascadeType.DETACH, CascadeType.REFRESH})
         private List<Pedido> listaPedidos;
      ```
    
   - El siguiente paso es crear una clase que guarde la lista de pedidos `GuardaListaPedidos.java` para un id de cliente dado que recuperaremos antes. 

   - Nota: para que nos funcionen los anteriores métodos de la nueva clase tendremos que añadirle que mapee también la clase `Pedido.java` en el fichero `hibernate.cfg.xml`
