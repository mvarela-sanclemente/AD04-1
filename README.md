# Mapear unha clase JAVA
As clases JAVA deben de ter as seguintes características:

- Deben ter un constructor público sen ningún tipo de argumentos.
- Implementar a interface *Serializable*.

Exemplo:

```java
import java.io.Serializable;

public class Equipo implements Serializable{
    
    private int id;
    private String nome;
    private String cidade;
    private int numeroSocios;
    
    public void Equipo(){
    }
    
    public Equipo(int id, String nome, String cidade, int numeroSocios){
        this.id = id;
        this.nome = nome;
        this.cidade = cidade;
        this.numeroSocios = numeroSocios;
    }
}
```

## Anotacións Hibernate

Incluímos as librerías de Hibernate e Maven de Maven no proxecto.

```xml
<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-core</artifactId>
    <version>5.4.10.Final</version>
</dependency>

<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.18</version>
</dependency>
```

A continuación vamos ver como podemos especificar con anotacións como se gardará esta na clase en táboas do modelo relacional.

```java
import java.io.Serializable;
import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.Id;
import javax.persistence.Table;

@Entity
@Table(name="Equipo")
public class Equipo implements Serializable{
    
    @Id
    @Column(name="id")
    private int id;
    @Column(name="nome")
    private String nome;
    @Column(name="cidade")
    private String cidade;

    @Column(name="numeroSocios")
    private int numeroSocios;
    
    public void Equipo(){
    }
    
    public Equipo(int id, String nome, String cidade, int numeroSocios){
        this.id = id;
        this.nome = nome;
        this.cidade = cidade;
        this.numeroSocios = numeroSocios;
    }
    
}
```
- A anotación **@id@ indica que é a clave principal da táboa.
- A anotación **@Column(name="nomeColumna")** é o nome da columna onde se gardará ese atributo.

## Configuración de Hibernate
Debemos configurar Hibernate para poder usalo. Para iso imos crear a clase HibernateUtil. nela indicaremos a configuración de Hibernate e engadiremos cales son as clases que se van a mapear.

```java
import java.util.Properties;
import org.hibernate.HibernateException;
import org.hibernate.SessionFactory;
import org.hibernate.boot.registry.StandardServiceRegistryBuilder;
import org.hibernate.cfg.Configuration;
import org.hibernate.cfg.Environment;
import org.hibernate.service.ServiceRegistry;

public class HibernateUtil {
    private static SessionFactory sessionFactory;
    
    //Este método devolve a sesión para poder facer operacións coa base de datos
    public static SessionFactory getSessionFactory(){
        
        //Se a sesion non se configurou, creamolo
        if(sessionFactory == null){
            try{
                Configuration conf = new Configuration();
                
                //Engadimos as propiedades
                Properties settings = new Properties();
                
                //Indicamos o conector da base de datos que vamos a usar
                settings.put(Environment.DRIVER,"com.mysql.cj.jdbc.Driver");
                
                //Indicamos a localización da base de datos que vamos a utilizar
                settings.put(Environment.URL,"jdbc:mysql://192.168.56.101:3306/hibernate");
                
                //Indicamos o usuario da base de datos con cal nos vamos conectar e o seu contrasinal
                settings.put(Environment.USER,"userhibernate");
                settings.put(Environment.PASS,"abc123.");
                
                //Indicamos o dialecto que ten que usar Hibernate 
                settings.put(Environment.DIALECT,"org.hibernate.dialect.MySQL5Dialect");
                
                //Indicamos que se as táboas todas se borren e se volvan crear
                settings.put(Environment.HBM2DDL_AUTO, "create-drop");
                
                //Indicamos que se mostre as operacións SQL que Hibernate leva a cabo
                settings.put(Environment.SHOW_SQL, "true");
                conf.setProperties(settings);
                
                //Engaidmos aquelas clases nas que queremos facer persistencia
                conf.addAnnotatedClass(Equipo.class);
                
                ServiceRegistry serviceRegistry = new StandardServiceRegistryBuilder().applySettings(conf.getProperties()).build();
                sessionFactory = conf.buildSessionFactory(serviceRegistry);
            }catch(HibernateException e){
                e.printStackTrace();
            }
        }
        return sessionFactory;
    }
    
}
```

## Como usar hibernate
Neste apartado vamos ver como gardar un obxecto da clase Equipo.

```java
import org.hibernate.HibernateException;
import org.hibernate.Session;
import org.hibernate.Transaction;

public class Main {
    
    public static void main(String args[]){
        //Creamos un equipo
        Equipo equipo = new Equipo(1, "San Clemente", "Santiago", 1000);
        Transaction tran = null;
        try{
            //Collemos a sesión de Hibernate
            Session session = HibernateUtil.getSessionFactory().openSession();
            //Comenzamos unha transacción
            tran = session.beginTransaction();
            
            //Gardamos o equipo
            session.save(equipo);
            
            //Facemos un commit da transacción
            tran.commit();
        }catch(HibernateException e){
            e.printStackTrace();
        }
    }
    
}
```

**Antes de executar, creamos a táboa 'hibernate' na base de datos.**
