# <center> Laboratory 2 </center>
# <center> Programming Distributed Applications </center>
## <center> Building a distributed system </center>

#### Microservices Are Distributed Systems
The original definition of a distributed system is: 
"A distributed system is a model in which components 
located on networked computers communicate and coordinate 
their actions by passing messages." (Wikipedia) And this is
exactly what happens in microservices-based architectures.

The individual services are deployed to cloud instances,
physically running somewhere as they exchange messages.
This is a big difference to how we used to build centralized applications. 
Instead of having a bunch of servers in our datacenter that 
handle all kinds of synchronization, transactions, and failover 
scenarios on our behalf, we now have individual services that evolve 
independently and aren't tied to each other. There are some fundamental
challenges that are unique to distributed computing. Among them are fault
tolerance, synchronization, self-healing, backpressure, network splits, and much more.

![](proxy.png)

To perform the task it is advisable to use the Docker container, Docker Compose.
DB Cassandra is proposed for the database role.</br>

**Docker** - Docker is a set of platform as a service products that use
OS-level virtualization to deliver software in packages called containers. 
Containers are isolated from one another and bundle their own software, 
libraries and configuration files; they can communicate with each other through well-defined channels.

**Docker Compose** - is used to run multiple containers as a single service. 
For example, suppose you had an application which required NGNIX and MySQL, 
you could create one file which would start both the containers as a service without the need to start each one separately.

**Cassandra** - Apache Cassandra is a free and open-source, distributed,
wide column store, NoSQL database management system designed to handle large
amounts of data across many commodity servers, providing high availability with no single point of failure. 

Because we have limited resources and lack of necessary operating system, our goal 
was to install Cassandra on our computer through a server instance and create two routes: one for reading and one for writing.

#### Installing the Cassandra instance on the local computer
![](cassandra.png) </br>
![](cassandra1.png) </br>
![](cassandra2.png) </br>
![](cassandra3.png) </br>

To create the read and write routes in the created tables we used the Java programming language
```
import com.datastax.driver.core.*;

        import java.time.Instant;
        import java.time.ZoneId;
        import java.util.Date;
        import java.util.UUID;

public class cassandra {

    private final static String KEYSPACE_NAME = "example_keyspace";
    private final static String REPLICATION_STRATEGY = "SimpleStrategy";
    private final static int REPLICATION_FACTOR = 1;
    private final static String TABLE_NAME = "example_table";

    public static void main(String[] args) {

        // Setup a cluster to your local instance of Cassandra
        Cluster cluster = Cluster.builder()
                .addContactPoint("localhost")
                .withPort(9042)
                .build();

        // Create a session to communicate with Cassandra
        Session session = cluster.connect();

        // Create a new Keyspace (database) in Cassandra
        String createKeyspace = String.format(
                "CREATE KEYSPACE IF NOT EXISTS %s WITH replication = " +
                        "{'class':'%s','replication_factor':%s};",
                KEYSPACE_NAME,
                REPLICATION_STRATEGY,
                REPLICATION_FACTOR
        );
        session.execute(createKeyspace);

        // Create a new table in our Keyspace
        String createTable = String.format(
                "CREATE TABLE IF NOT EXISTS %s.%s " + "" +
                        "(id uuid, timestamp timestamp, value double, " +
                        "PRIMARY KEY (id, timestamp)) " +
                        "WITH CLUSTERING ORDER BY (timestamp ASC);",
                KEYSPACE_NAME,
                TABLE_NAME
        );
        session.execute(createTable);

        // Create an insert statement to add a new item to our table
        PreparedStatement insertPrepared = session.prepare(String.format(
                "INSERT INTO %s.%s (id, timestamp, value) values (?, ?, ?)",
                KEYSPACE_NAME,
                TABLE_NAME
        ));

        // Some example data to insert
        UUID id = UUID.fromString("1e4d26ed-922a-4bd2-85cb-6357b202eda8");
        Date timestamp = Date.from(Instant.parse("2020-01-01T01:01:01.000Z"));
        double value = 123.45;

        // Bind the data to the insert statement and execute it
        BoundStatement insertBound = insertPrepared.bind(id, timestamp, value);
        session.execute(insertBound);

        // Create a select statement to retrieve the item we just inserted
        PreparedStatement selectPrepared = session.prepare(String.format(
                "SELECT id, timestamp, value FROM %s.%s WHERE id = ?",
                KEYSPACE_NAME,
                TABLE_NAME));

        // Bind the id to the select statement and execute it
        BoundStatement selectBound = selectPrepared.bind(id);
        ResultSet resultSet = session.execute(selectBound);

        // Print the retrieved data
        resultSet.forEach(row -> System.out.println(
                String.format("Id: %s, Timestamp: %s, Value: %s",
                        row.getUUID("id"),
                        row.getTimestamp("timestamp").toInstant().atZone(ZoneId.of("UTC")),
                        row.getDouble("value"))));

        // Close session and disconnect from cluster
        session.close();
        cluster.close();
    }
}
```
