Movies Example Application
==========================

How to use Spring Boot, Spring Data, and Neo4j together.

Spring Data Neo4j enables convenient integration of Neo4j in your Spring-based application.
It provides object-graph mapping (OGM) functionality and other features common to the Spring Data projects.

[NOTE]
*This project uses Spring Data Neo4j 5.*
It is optimized for working with Neo4j Desktop and based on Neo4j's query language, Cypher.

The example project is described in detail on the https://neo4j.com/developer/example-project/[Neo4j Developer Site]

== Quickstart

. http://neo4j.com/download[Download, install, and start Neo4j Desktop].
. From here, there are two ways you can access the end points.
    1) Open the browser web interface at http://localhost:7474
        a. Configure a username and password, if you haven't already.
    2) In Neo4j Desktop, click on your project, click on 'Manage' on the database you are using, click 'Open Browser'.
        a. Should not need to log in. Password was set up when you set up the database.
. Run `:play movies` command, and click and run the Cypher statement to insert the dataset
. Clone this project from GitHub
. Update `src/main/resources/application.properties` with the username and password you set above.
. Run the project with `mvn spring-boot:run`.

== Code Walkthrough

To use Neo4j with Spring Data Neo4j, you just add the dependency for https://projects.spring.io/spring-boot/[Spring-Boot] and https://projects.spring.io/spring-data-neo4j/[Spring-Data-Neo4j] to your build setup.

.pom.xml
[source,xml]
----
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-neo4j</artifactId>
</dependency>
----
include::pom.xml[tags=dependencies]

Annotate your `@NodeEntity` and `@RelationshipEntity`. You can use the OGM `Session` to access Neo4j APIs and object graph mapping functionality.

.Movie.java
[source,java]
----
@NodeEntity
public class Movie {

    @Id
    @GeneratedValue
    private Long id;

    private String title;

    private int released;

    private String tagline;

    @Relationship(type = "ACTED_IN", direction = Relationship.INCOMING)
    private List<Role> roles;
}
----
include::src/main/java/movies/spring/data/neo4j/domain/Movie.java[tags=movie]

Additionally, you can leverage the convenient Spring-Data repositories to get interface-based DAO implementations injected into your Spring Components.

.MovieRepository.java
[source,java]
----
public interface MovieRepository extends Neo4jRepository<Movie, Long> {

    Movie findByTitle(@Param("title") String title);

    Collection<Movie> findByTitleLike(@Param("title") String title);

    @Query("MATCH (m:Movie)<-[r:ACTED_IN]-(a:Person) RETURN m,r,a LIMIT {limit}")
    Collection<Movie> graph(@Param("limit") int limit);
}
----
include::src/main/java/movies/spring/data/neo4j/repositories/MovieRepository.java[tags=repository]

In our case, we use the repository from a `MovieService` to compute the graph representation for the visualization.
The service is then injected into our main Boot application, which also doubles as `@RestMvcController` which exposes the `/graph` endpoint.

The other two endpoints for finding multiple movies by title and loading a single movie are provided out of the box by the https://projects.spring.io/spring-data-rest/[Spring-Data-Rest project], which exposes our `MovieRepository` as REST endpoints.

The rendering of the movie objects (and related entities) happens automatically via Jackson mapping.

== The Stack

These are the components of our Web Application:

* Application Type:         Spring-Boot Java Web Application (Jetty)
* Web framework:            Spring-Boot enabled Spring-WebMVC, Spring-Data-Rest
* Persistence Access:       Spring-Data-Neo4j 5.0.5
* Database:                 Neo4j-Server 3.3.x
* Frontend:                 jquery, bootstrap, http://d3js.org/[d3.js]

== Endpoints:

Get Movie

To run our setup queries through our REST endpoints:

From the Neo4j Desktop main console, go to 'Manage' on your database, then choose the 'Terminal' tab. You can copy/paste the commands below into that terminal and should see results.

You can also go to http://localhost:8080 to interact with the data.

Now feel free to add other queries to your application to see more data and relationships!

----
// JSON object for single movie with cast
curl http://localhost:8080/movies?title=The%20Matrix

// list of JSON objects for movie search results
curl http://localhost:8080/movies?title=*matrix*

// JSON object for whole graph viz (nodes, links - arrays)
curl http://localhost:8080/graph
----
