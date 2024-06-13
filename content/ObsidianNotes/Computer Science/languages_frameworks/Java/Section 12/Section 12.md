**⇒ This section will focus on database - JDBC**

Working with databases can be done from the terminal, and really, is accessible via checking documentation. Now, we will look at working with JDBC API, which will then allow us to work with a database from a Java application. JDBC is simply a middleman application that sits between the database level and the Java application. To use a particular data source from an application, we will need JDBC driver for that data source: **something that is provided by all main database applications → will need to download the driver.**

- It allows us to execute SQL queries through the Java application

  

We’ll look at SQLite, but these can be then applied to any database.

  

### SQLite Commands

```SQL
// useful 
.headers on // just prints the column labels for better viewing 
// to see tables in the DB
.tables 

// to see command that created table
.schema 

// see everything
.dump

// creating a table 
create table contacts (name text, phone integer, email text);

// inserting into a table 
insert into contacts(name, phone, email) values ('Allan', 6472080565, 'a7yin@uwaterloo.ca');

// retrieving 
SELECT * FROM contacts // this returns all entries in the database 

// sepecific retrieval 
SELECT phone, email FROM contacts WHERE name="Allan";

// updating 
UPDATE contacts SET email="new_email@gmail.com"; // this is global, changes the email for all entries in the database. Often times, not what we want. Can specify. 

// updating with WHERE 
UPDATE contacts SET email="new_email@gmail.com" WHERE name="Steve";

// Delete entry 
DELETE FROM contacts WHERE phone=6472080565; // need where, without it, might delete everything 
```

  

**⇒ Order by and Joins**

```SQL
// from just selecting, ordered by primary key. If no primary key, undefined order. To specify something specific, we can do:
SELECT * FROM artists ORDER BY name; // can add NOCASE to be ordered without case being considered 

SELECT * FROM artists WHERE _id=133; // returns all entries in the artists table that have the _id 133 (so, only one)
```

What if we want to select something that is part of multiple tables? We will use the join clause:

```SQL
SELECT songs.track, songs.title, albums.name FROM songs JOIN albums ON songs.album = albums._id
```

![[Screenshot_2023-04-30_at_11.29.20_AM.png]]

This query should return the song tracks, titles and album names where, from their respective tables, we have the condition `songs.album = albums._id` to be true. There are multiple types of joins, and really, the join above that was demonstrated is a use of `INNER JOIN`. So, the above is the same thing as:

```SQL
SELECT songs.track, songs.title, albums.name FROM songs INNER JOIN albums ON songs.album = albums._id
```

Here are the other ones:

- Left Join: The `LEFT JOIN` keyword returns all records from the left table (table1), and the matching records from the right table (table2). The result is 0 records from the right side, if there is no match.
- Right join: The `RIGHT JOIN` keyword returns all records from the right table (table2), and the matching records from the left table (table1). The result is 0 records from the left side, if there is no match.
- Full join: The `FULL OUTER JOIN` keyword returns all records when there is a match in left (table1) or right (table2) table records.

  

Now, lets look at JDBC. We will need to first install the database driver, and add it to the application. The setup looks like this:

```SQL
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;
import java.sql.Statement;

public class Main {
    public static void main(String[] args) {
        try {
            Connection conn = DriverManager.getConnection("jdbc:sqlite:/Users/allanyin/Documents/CS-TOOLS/Java/JDBC/testjava.db");
            // will need to close this resource. We can also just do a try with resources, which will close the resource for us
            // conn.setAutoCommit(false); // this will make it so, db not automatically ujpdated after executing line here. Will need explicit call
            Statement statement = conn.createStatement();
            // statement.execute("CREATE TABLE contacts (name TEXT, phone INTEGER, email TEXT)");
            // connection string, this returns connection instance
            // this is how we connect to a database
            statement.execute("INSERT INTO contacts (name, phone, email) VALUES ('Allan', 123, 'allan@a.com')");
            statement.execute("INSERT INTO contacts (name, phone, email) VALUES ('Brian', 456, 'brian@a.com')");
            statement.execute("INSERT INTO contacts (name, phone, email) VALUES ('Pierre', 789, 'pierre@a.com')");

            statement.close();
            conn.close();
            // order of closing matters
        } catch (SQLException e) {
            System.out.println(e.getMessage());
        }
    }
}
```

We create an initial connection, and a statement object, in which we can then make changes to the DB. For insertion, updating, and deleting, we just simply `execute()` the SQL query. For selecting, we can get those results from the database into the Java program through:

```SQL
statement.execute("SELECT * FROM contacts");
ResultSet results = statement.getResultSet();
while (results.next()) {
	// do whatever, we have the results
}
results.close();

// we can use just one function: 

ResultSet results = statement.executeQuery("SELECT * FROM contacts");
```

Now, for actual projects, it will make sense to define global constants (for things such as table names, column names, etc.). In addition, it will make sense, and be more practical, to perform these initializations inside of a class, like so:

```SQL
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;

public class Datasource {

    public static final String DB_NAME = "music.db";

//    public static final String CONNECTION_STRING = "jdbc:sqlite:D:\\databases\\" + DB_NAME;
    public static final String CONNECTION_STRING = "jdbc:sqlite:/Volumes/Production/Courses/Programs/JavaPrograms/Music/" + DB_NAME;

    public static final String TABLE_ALBUMS = "albums";
    public static final String COLUMN_ALBUM_ID = "_id";
    public static final String COLUMN_ALBUM_NAME = "name";
    public static final String COLUMN_ALBUM_ARTIST = "artist";

    public static final String TABLE_ARTISTS = "artists";
    public static final String COLUMN_ARTIST_ID = "_id";
    public static final String COLUMN_ARTIST_NAME = "name";

    public static final String TABLE_SONGS = "songs";
    public static final String COLUMN_SONG_TRACK = "track";
    public static final String COLUMN_SONG_TITLE = "title";
    public static final String COLUMN_SONG_ALBUM = "album";

    private Connection conn;

    public boolean open() {
        try {
            conn = DriverManager.getConnection(CONNECTION_STRING);
            return true;
        } catch(SQLException e) {
            System.out.println("Couldn't connect to database: " + e.getMessage());
            return false;
        }
    }

    public void close() {
        try {
            if(conn != null) {
                conn.close();
            }
        } catch(SQLException e) {
            System.out.println("Couldn't close connection: " + e.getMessage());
        }
    }
}

// we will then, simply work with the DataSource object in the main Java file
```