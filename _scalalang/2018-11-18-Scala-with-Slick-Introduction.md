---
title: "Scala and Slick - An Introduction"
collection: scalalang
permalink: /scalalang/scalaandslick
---

<figure>
	<img src="/images/scala_slick.png">
</figure>

In this article, I am going to explain working with Scala and slick, this is a tutorial to help beginners get a hang of slick as well as introduce to the concept of Scala Programming.

Why I love scala is because of its expressiveness, it allows developers to program with utmost ease.

Slick and it's properties that make it awesome
Slick allows you to do database query as though you are working with Scala collections i.e it gives you a bonus point of being able to run `map`, `filter`, `flatmap` etc. on your stored data.

Apart from this, you get compilation Type Checks, hence you don't have to go into running your application with some hidden bugs.

Also, you get a non-blocking request as part of the bonus for using slick, as slick uses a DBIO Monadic API, you are able to chain operations that you can later flatten out

Let's put this into an example fast, say you have a list and you can apply map to generate a list of list as shown below.

<script src="https://gist.github.com/adekunleba/3c111a205c5dfd0477bc9ea15a242d75.js">
</script>

The above `List[Int]` produces a `List[List[Int]]` as per our example, `flatmap` flattens out the inner List, hence you don't have to do `map` then `flatten`. So, let's say you do a DB query that can return with map `List[List[SomeType]]`, because of the monadic API in slick you get a `flatmap` over many of your queries.

Now that we have established what benefits we can get with Slick let's do a quick rundown of an example. We will need a db for this, you can use anyone be it MYSQL, PostGres, MariaDB. I am biased towards MariaDB hence our example will be using mariadb. To set up Mariadb, you can refer to [this article](https://linuxize.com/post/how-to-install-mariadb-on-ubuntu-18-04/). 

Now that we have mariadb installed we can start writing our scala application to interact with mariadb.

Since we will get a free introduction in this article as we intend to build something close to what a real-life software engineer get's in practice for this example, I like to use one of scala seed project to kickstart my scala projects.

```scala
sbt -Dsbt.version=0.13.15 new akka/akka-http-quickstart-scala.g8
```
Once you fill out the necessary information, including package name, version of Akka and name of the project, you can import the project into Intelij.

This is as simple as open Intelij => Files => Open and navigate to your newly created file.

This seed project gives you a lot free of charge which we will just basically extend for our case since our main aim is to use slick with this minimal web service. 

#### Slick and Scala 

To use slick in your project, you need to add slick as dependency and also add the driver for the database you are using, in our case we are using Mariadb hence mysql driver it is, for postgres you can get the driver from Maven Repository.
Finally, our `build.sbt` should look like this:

```scala
libraryDependencies ++= Seq(

      //Akka packages
      "com.typesafe.akka" %% "akka-http"            % "10.1.5",
      "com.typesafe.akka" %% "akka-http-spray-json" % "10.1.5",
      "com.typesafe.akka" %% "akka-http-xml"        % "10.1.5",
      "com.typesafe.akka" %% "akka-stream"          % "2.5.18",

      //Database packages
      "com.typesafe.slick" %% "slick"               % "3.2.1",
      "com.typesafe.slick" %% "slick-hikaricp"      % "3.2.1",
      "mysql"              % "mysql-connector-java" % "5.1.34",

      //Migration tool
      "org.flywaydb"       %  "flyway-core"         % "5.0.7",

      //Test packages
      "com.typesafe.akka" %% "akka-http-testkit"    % "10.1.5" % Test,
      "com.typesafe.akka" %% "akka-testkit"         % "2.5.18"     % Test,
      "com.typesafe.akka" %% "akka-stream-testkit"  % "2.5.18"     % Test,
      "org.scalatest"     %% "scalatest"            % "3.0.5"         % Test,
)
```

Next we will like to create our own Registry Actors and Routes that integrates with slick, hence delete the files `UserRegistoryActor.scala` and `UserRoutes.scala` in your seed project.

*Side Notes:* The **Registry actors** are basically actors that help you manage your requests so that you have a non-blocking and concurrent web-service, while **Routes** are basically you web service address

Once we have this, let's do a quick configuration for the db to connect to our application. You keep your configuration files in `application.conf` in resources folder:
Our sample configuration is as shown below:

```scala
database {
  url = "jdbc:mysql://localhost:3306/sampledb?useUnicode=true&characterEncoding=UTF-8"
  driver = "com.mysql.jdbc.Driver"
  user = "root"
  password = "usedbsetupduringinstallatonofdb"
  numThreads = 4
  maxConnections = 8
  minConnections = 2
  registerMbeans = true

  numberOfThreads = 10
}
```

Also, it's good to use a migration package since you still need to have a mapping of the representation of your DB columns when slick wants to create a new table in your db 

We are using `flyway` package to handle Data base migration. For anyone with experience with python, flyway is just like alembic. One you had the package into your sbt file we should be fine.

Next will be to create an sql for your table, you can add every new version of your sql update in your resource folder. Also, there is an important way in naming your sql file this is to preserve versioning, the syntax for naming is `VX__Information.sql` where `VX` is the latest version of your migration and its changes, and `Information` is basically what the version is adding. A concrete naming example is `V1__Create_user_table.sql`, a sample of such file is as shown below:

```sql

-- -----------------------------------------------------
-- Table `users`
-- -----------------------------------------------------
CREATE TABLE IF NOT EXISTS `users` (
`id` BIGINT(20) NOT NULL AUTO_INCREMENT,
`username` VARCHAR(200) NOT NULL,
`password` VARCHAR(300) NOT NULL,
`location` VARCHAR(200) NOT NULL,
`gender` INT(4) NOT NULL,
PRIMARY KEY (`id`),
UNIQUE INDEX `username_UNIQUE` (`username` ASC))
ENGINE = InnoDB;

```


After this we can go ahead and write our `flyway` and `database` configuration in our scala app, so that we can write our first migration.

It will be good to wrap our general db migration in a Trait since both flyway and database configuration shares attributes from it

```scala

//Load Configuration files

import com.typesafe.config.ConfigFactory

trait Config {
      
      //set's up ConfigFactory to read from application.conf
      private val config = ConfigFactory.load() 
      
      //Get configurations key vales for database
      private val databaseConfig = config.getConfig("database")

      val databaseUrl = databaseConfig.getString("url")
      val databaseUser = databaseConfig.getString("user")
      val databasePassword = databaseConfig.getString("password")
}
```

In another file for `Migration.scala`, we then have the flyway migration, also can be wrapped in a Trait since its methods and properties will be used in a later phase.

```scala

import org.flywaydb.core.Flyway

trait MigrationConfig extends Config {

      private val flyway = new Flyway()

      flyway.setDataSource(databaseUrl, databaseUser, databasePassowrd)

      //I am adding this in case you make a mistake in your migration script, 
      //do a flyway.repair() before you migrate again
      //flyway.repair()

      def migrate() = flyway.migrate()
      def reloadSchema() = {
            flyway.clean()
            flyway.migrate()
      }
}
```

Now we should also create this configuration for our database

```scala
trait DatabaseConfig extends Config {

      val driver = slick.jdbc.MySQLProfile

      import driver.api._

      def db = Database.forConfig("database")
      //We make database session implicits so that it will
      //be always be available for any class that extends Database Config.
      implicit val session: Session = db.createSession()
}
```
Now that we have all this set up we can now create our `TableQuery` for our user SQL file and also implement the various the operations we will be applying on the users table. **TableQuery** class basically allows you to map your Table pattern in your code to your database table so that we continue to write db queries as scala syntax and we get the promise of writing queries as scala collection which slick promised us.

Therefore, our `UserTable.scala` looks like this:

```scala
import com.akkaactors.db.models.User
import slick.jdbc.MySQLProfile.api._

//We can define a type alias for userID  
package object definition {
      type UserId = Long
}

//Also we need a case class that act as a place holder for this column table
case class User(id: Option[UserId], username: String, password: String, location: String, gender: Int)



class UsersTable(tag: Tag) extends Table[User](tag, "users") {

  def id = column[UserId]("id", O.PrimaryKey, O.AutoInc)
  def username = column[String]("username")
  def password = column[String]("password")
  def location = column[String]("location")
  def gender = column[Int]("gender")

  //Add id to *
  def * = (id.?, username, password, location, gender) <> ((User.apply _).tupled, User.unapply)
}
```
The case class is important since it's what connects other parts of our application with the table name in db i.e your `User` case class now embodies the columns of your user table in db hence you basically can apply you scala functions on `User` as though they are columns in your db.

To define the operations on your user table we create another file `UsersDao.scala`. Also, usually for every query to db with slick, you need to do a `db.run()` to execute the query, you can basically abstract away that portion by providing it as an implicit parameter such that when you just call your operations, you app picks it up as an action and implement `db.run(operation)` on it without you having to always call db.run().
```scala
import slick.jdbc.MySQLProfile.api._

import scala.concurrent.Future

trait BaseDao extends DatabaseConfig {

      val usersTable = TableQuery[UsersTable]


      protected implicit def executeFromDb[A](action: SqlAction[A, NoStream, _ <: slick.dbio.Effect]): Future[A] = {
    db.run(action)
  }

}

object UsersDao extends BaseDao {

  def findAll: Future[Seq[User]] = usersTable.result
  def create(user: User): Future[UserId] = usersTable.returning(usersTable.map(_.id)) += user
  def findById(userId: UserId): Future[User] = usersTable.filter(_.id === userId).result.head

  def delete(userId: UserId): Future[Int] = usersTable.filter(_.id === userId).delete
}
```

With this done, we are almost done, we can basically now be making our requests and interacting with db based on whatever it is we want to do.
So our toy example will require me to write my service that once you send to your web address, you get to use actors to perform non-blocking operations on the request. So we now have Actor for this. The remaining code is based more on implementing a service with akka-http, you can get a glimpse into the very fundamental of akka-http in one of my article [here](https://medium.com/@Babatee760/simple-web-api-with-akka-http-and-redis-database-f9b3826f711a). I basically extended the approach in the article to leverage the use of Actor, so that we move towards a concurrent approach to developing web services.

Let's create our own `UserRegistryActor.scala`

```scala
object UserRegistryActor{
  final case class ActionPerformed(description: String)
  final case object GetUsers
  final case class CreateUser(user: User)
  final case class GetUser(name: UserId)
  final case class DeleteUser(name: UserId)

  //This is you registering the Actor
  def props: Props = Props[UserRegistryActor]
}

class UserRegistryActor extends JsonSupport with Actor with ActorLogging {
  import UserRegistryActor._
  import context.dispatcher
  //DB Implementation here

  def receive: Receive = {

   //Get all users
    case GetUsers =>
      val mysender = sender
      val allUsers = UsersDao.findAll
      allUsers.onComplete {
        case Success(usr) => mysender ! Users(usr)
        case Failure(failureUsr) => println("Data not found to find all Users in Database")
      }

   //Create User
    case CreateUser(user) =>
      UsersDao.create(user)
      sender() ! ActionPerformed(s"User ${user.username} created.")

    //Get a particular User
    case GetUser(id) =>
      val user = UsersDao.findById(id)
      val userSender = sender
      user.onComplete {
        case Success(usr) => userSender ! usr
        case Failure(failureUsr) => println(s"$id user not found")
      }

   //Delete a user based on an id
    case DeleteUser(id) =>
      val user = UsersDao.delete(id)
      val delSender = sender
      user.onComplete {
        case Success(del) => delSender ! ActionPerformed(s"User $id deleted")
        case Failure(delUser) => println(s"Unable to Delete user $id")
      }
  }
}
```


Finally, the entity that connects your client to your backend service is the routes. So we will implement the routes in `Routes.scala`

```scala

  //Json Support is important for marshalling your requests either json, xml into a case class with akk-http
   trait UserRoutes extends JsonSupport{

         implicit def system: ActorSystem

         def userRegistryActor: ActorRef

         //Actor ask needs this to ever function
         implicit lazy val timeout = Timeout(5.seconds)


         def userRoutes: Route = pathPrefix("users") {
            pathPrefix("enroll-user") {pathEnd {
                  concat(
                  get {
                           val users: Future[Users] =
                           (userRegistryActor ? GetUsers).mapTo[Users]
                           complete(users) },
                  post { entity(as[User]) { user =>
                            val userCreated = (userRegistryActor ? CreateUser(user)).mapTo[ActionPerformed]
                            onSuccess(userCreated) { performed =>
                                   log.info(s"Created User [${user.username}]: ${performed.description}")
                            complete(StatusCodes.Created, performed.description)}}})
                     } ~ 
                  path(IntNumber) { id =>
                  get {
                        val maybeUser: Future[User] = (userRegistryActor ? GetUser(id)).mapTo[User]
                        rejectEmptyResponse {complete(maybeUser)}
                  } ~
                  delete {
                        val userDeleted: Future[ActionPerformed] = (userRegistryActor ? DeleteUser(id)).mapTo[ActionPerformed]
                        onSuccess(userDeleted) { performed =>
                        log.info(s"Deleted user $id", performed.description)
                        complete(StatusCodes.OK, performed)
                        }
                     }
                  }
               }
         }
      }

```
So with this we have built a minimal backend, we can scale this by basically adding more operations based on request and what we intend to achieve, also we can interact with more tables on our DB, by basically following the same approach of creating TableQuery.

You can play around with the codes for this article by cloning the [github repository](https://github.com/adekunleba/sample-slick)
