---
layout: post
title:  "PostgreSQL and .NET"
date:   2015-02-28
categories: Postgres dotnet
---

I used to like ORMs just fine but nowadays I find that I'm more of an advocate of doing much or all of the database work yourself. There's plenty of reasons I think this, but that's not really what this post is about. I just think there's a lot to be said for the extra control not using a full blown ORM grants you. (Though I do still like using Micro ORMs as you'll see)

These past few weeks I've been looking at PostgreSQL, and I love it. It's simple to set up, and it's simple to use. Coming from a SQL Server background it feels almost liberating. (seriously, that 12 step process...).

Up to yet I've only really been messing with Postgres by executing SQL queries against the database directly using the admin tools, but since I do a lot of work in the .NET space I figured it was time to give it a good go from that perspective. As it turns out, it's easy, frighteningly so.

Installing Postgres on Windows is crazy simple. No insane 12 step process, no signup, no installing of an installer. I won't cover the installation process here, I want to keep this post focused on just using Postgres in .NET. There are plenty of guides out there, or you could check out the documentation.

Once you do have Postgres installed and set up were ready to get started writing some code.

## Creating a Database

I won't go into too much detail on the inner workings of postgres here (documentation). I'll assume you already know how to execute queries against it using PGAdmin or the command line.

I'm going to create a simple one table database in Postgres that we can write queries against in the next couple of sections. If you're already used to writing SQL none of this will be particularly new to you.

```
CREATE DATABASE PostgresDotNet;

-- Make sure you connect to this database after creating it!
-- As far as I'm aware there is no USE statement in Postgres

CREATE TABLE Avengers (
  Id serial primary key not null,
  Name varchar(255) not null,
  AlterEgo varchar(255)
);

INSERT INTO Avengers(Name, AlterEgo)
VALUES
  ('Captain America', 'Steve Rogers'),
  ('Iron Man', 'Tony Stark'),
  ('Hawkeye', 'Clint Barton'),
  ('Hulk', 'Bruce Banner'),
  ('Spider Man', 'Peter Parker'),
  ('Thor', 'Thor Odinson');
```

First we're creating a database called PostgresDotNet, then after switching to this database we are creating a table on it named Avengers, with an Id field, a Name field, and an AlterEgo field. Then we're inserting a bunch of values into the table. Nothing too special, probably nothing here that you havent seen before in other flavours of SQL.

Just to make sure everything has worked correctly let's get all of the data we just entered back.

```
SELECT *
FROM Avengers;
```

You should see all of the data we just entered, only with Id's that have been automagically generated for us because we gave the Id field a data-type of serial.

Now we have a database and table set up let's head over to Visual Studio and write some queries!

## Working with ADO.NET Directly
Booo! Hiss! ADO.NET!? What is this? The early 2000's?

Well, yes, it is. There's still 985 years to go. Also, ADO.NET is still used by people in production to this day because it's fast and provides a great level of control. If you don't like it that's fine... There are other options and abstractions that you can lay on top of this later. I just think it's a good idea to know how to work with it at this level before moving on. If you've never written this kind of ADO.NET code before, don't worry, it's really very simple. If a bit tedious.

## Bringing in the relevant NuGet packages
All of the Postgres Connection magic is brought to us using a NuGet packaged called Npgsql. It works just like the old ADO.NET code you're probably used to writing.

You can install it via the package manager console by typing 
Install-Package Npgsql

This will add all of the necessary references and DLLs into your project.

## Setting up the web.config/app.config file
Next we need to tell the project about our database, again this is very similar to how we do it when working with SqlServer.

```
<connectionStrings>
  <add name="PostgresDotNet"
       connectionString="server=localhost; user={your username}; password={your password}; database=postgresdotnet"
       providerName="Npgsql" />
</connectionStrings>
```

Not much new here right? The server will be localhost, this is where postgres is listening for requests. You'll also need to specify a username and password for the connection to use, and you'll also need to specify which database to connect to.

The main difference here is the providerName, you'll need to use Npgsql instead of System.Data.SqlClient which you may be more accustomed to.

## Creating a DbConnection
Okay over in our C# code we need to grab this ConnectionString using the ConfigurationManager, and store that in a variable.

```
string ConnectionString =
    ConfigurationManager.ConnectionStrings["PostgresDotNet"].ConnectionString;
```

Next we need to create a SqlConnection, or in our case a PgsqlConnection, using the ConnectionString we have specified.

```
using(NpgsqlConnection conn = new NpgsqlConnection(ConnectionString)) {
  conn.Open();
}
```

Remember to open the connection! and also wrap the whole thing in a using statement so that the connection is properly closed and disposed of when were done with it.

## Creating and Executing a SqlCommand
Next we can write our SQL Statement and store that in a variable, and create a NpgsqlCommand passing in both our SQL Statement and NpgsqlConnection.

After that we create a NpgsqlDataReader object by calling the ExecuteReader method on our command, and simply output the results of our reader to the console.

```
using(NpgsqlConnection conn = new NpgsqlConnection(ConnectionString)) {
  con.Open();
  const string sql = "SELECT Id, Name, AlterEgo FROM Avengers;";
  NpgsqlCommand cmd = new NpgsqlCommand(sql, conn);
  NpgsqlDataReader reader = cmd.ExecuteReader();

  while(reader.Read()) {
    Console.WriteLine(reader["name"]);
  }
}
```

All pretty much standard ADO.NET really, if you've used it to connect to SQL Server in the past you'll be right at home here.

Run your program to see a list of names from our database, did you notice how lightning quick it is? That's why people still like using ADO.NET!

Creating and Executing a Parametized SqlCommand
Most of the time you're going to want to execute parametized queries, unsuprisingly you do this the same way as you would if you were connecting to a SQL Server. By using AddWithValue() on your NpgsqlCommand.

```
using(NpgsqlConnection conn = new NpgsqlConnection(ConnectionString)) {
  con.Open();
  const string sql = "SELECT Id, Name, AlterEgo FROM Avengers WHERE Id = @id;";
  NpgsqlCommand cmd = new NpgsqlCommand(sql, conn);
  cmd.Parameters.AddWithValue("@id", 5);
  NpgsqlDataReader reader = cmd.ExecuteReader();

  while(reader.Read()) {
    Console.WriteLine(reader["name"]);
  }
}
```

## Using a Micro-ORM
Using a Micro-ORM is my preferred method of working with databases in .NET, I find they abstract just enough nastiness away that I can still be productive, while also having all the control I wan't over my queries. Which a full-blown ORM like Entity Framework just doesn't give me.

There's a few of them out there that you can choose from, like Biggy, Dapper, 
and even one that I wrote myself which you shouldn't use.

My Micro-ORM of choice is Dapper. I've been using it for a while and have yet to run into any real problems with it. It also works great with Postgres.

So let's do the same as we did above, only this time with help from Dapper...

## Setting up
Set up is the same fare as before, you'll need the Npgsql NuGet package again, along with dapper. As before you can install this in the package manager console by typing in Install-Package dapper

Next you'll need to add the connection string to the web.config file, exactly the same as before.

Finally, create an NpgsqlConnection object using the connection string. Just like before.

## Executing a Query
When working with MSSQL Dapper extends the SqlConnection object, and provides you with some helpful methods like Query and Execute, you have the option of also executing typed methods, which will perform object mapping for you. For more info on the inner workings of dapper take a look at the Github Repo.

Most of what you will be reading in the documentation also applies to PostgreSQL, the NpgsqlConnection object is extended to include the same methods that perform the same tasks.

So if we wanted to grab all of the data from a table like before we simply do the following:

```
using(NpgsqlConnection con = new NpgsqlConnection(ConnectionString)) {
  con.Open();
  const string sql = "SELECT Id, Name, AlterEgo FROM Avengers;";
  var results = con.Query(sql);

  foreach(var avenger in results) {
    Console.WriteLine(avenger.name);
  }
}
```

What if we have a type and want to map the results to it? Well in that case you execute .Query<T>() instead of .Query().

```
public class Avenger {
  public int Id { get; set; }
  public string Name { get; set; }
  public string AlterEgo { get; set; }
}

// ...

using(NpgsqlConnection con = new NgsqlConnection(ConnectionString)) {
  con.Open();
  const string sql = "SELECT Id, Name, AlterEgo FROM Avengers;";
  var results = con.Query<Avenger>(sql);

  foreach(var avenger in results) {
    Console.WriteLine(avenger.Name);
  }
}
```

The query methods that we have been using return an IEnumerable<T>, which means you're free to perform LINQ operations on them, cast them to lists, anything you need. Hopefully you've also noticed that it's fast almost as fast as using raw ADO.NET, unlike other ORMs I can name.

## Creating and Executing a Parametized SqlCommand
Executing Parametized SQL Commands is also pretty easy using dapper:

```
var ConnectionString = ConfigurationManager.ConnectionStrings["PostgresDotNet"].ConnectionString;

using (NpgsqlConnection con = new NpgsqlConnection(ConnectionString))
{
    con.Open();
    const string sql = "SELECT Id, Name, AlterEgo FROM Avengers WHERE Id = @id";
    DynamicParameters queryParams = new DynamicParameters();
    queryParams.Add("@id", 1);
    var results = con.Query<Avenger>(sql, queryParams);

  foreach (var avenger in results)
  {
      Console.WriteLine(avenger.Name);
  }
}
```

Well, there you have it. I'm very glad to see just how easy switching to Postgres from something like MSSQL really can be in .NET. There's almost no learning curve at all! (provided you're used to working with ADO.NET).