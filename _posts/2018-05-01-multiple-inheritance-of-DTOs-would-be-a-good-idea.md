---
layout: default
title: "Multiple inheritance of DTOs would be a good idea"
date:   2018-05-01 00:00:00 +0300
---

# {{ page.title }}

C# 8 is to have default interface implementations as one of it's new features
(as it's described e.g. [here](https://stackify.com/csharp-8-features/)).

This feature is described as reasonable and even exciting
(as most of the features shipped with every new C# version IMO).
The feature reminds good old .h C++ files
(with the ability to either have an implementation or not) --
everything new is well-forgotten old indeed.

The feature is also frighteningly reminiscent of another rather arguable C++ concept --
the concept of multiple inheritance and questions like "what implementation will it choose".
It's really a bit disturbing as it leads to one of the following:
* unclearness, ambiguity, and obligation to know more compiler implementation specifics
(which could be interesting, but yet redundant);
* cumbersome syntax - e.g. an obligation to specify a concrete interface
to take a method signature (with implementation) from.

Anyway -- time will tell.

I'm telling you all this as I think there is a reasonable application
for multiple inheritance in modern languages (especially backend ones).

## Use case

There is a common task, solution of which is still debated --
that is [object-relational mapping](https://en.wikipedia.org/wiki/Object-relational_mapping)
and whole [ORM vs Micro ORM debate](http://gunnarpeipman.com/2017/05/micro-orm/).
The most popular ORM in .NET is [Entity Framework (EF)](https://www.nuget.org/packages/EntityFramework/)
(just look at how many downloads it has gotten). Its main advantage is the ability to execute
database queries that are written in strongly typed language (e.g. C# LINQ) which leads to much better
refactoring, unit testing, and logic versioning. But one of its main disadvantages (especially for beginners
striving to write both efficient and good code) is it's complicated object graphs (especially the detached ones),
which lead to DTO classes like the following:

```csharp
public class Person
{
  public Guid PersonId { get; set; }
  public Guid Name { get; set; }

  public Employee Employee { get; set; }
}
public class Employee
{
  public string Position { get; set; }

  public Person Person { get; set; }
}
```

which correspond to the respective Person and Employee database tables connected with one-to-zero-or-one relation.

And a typical DB queries looks like:

```csharp
Person person = UnitOfWork.Persons.FirstOrDefault(p => p.PersonId == wantedPersonId);
// suppose the query is set to join and map the respective person
Employee employee = UnitOfWork.Employees.FirstOrDefault(e => e.PersonId == wantedPersonId);

// is true as EF holds a graph of fetched and mapped objects and takes references from there
bool doWeHaveAGraph = employee.Person == person;

// one way of updating ...
person.Name = newName;
UnitOfWork.Save();

// ... and the other one that does completely the same
employee.Person.Name = newName;
UnitOfWork.Save();
```

I already mentioned that graphs could be detached (i.e. fetched objects will not be in a graph --
see [tracking](https://docs.microsoft.com/en-us/ef/core/querying/tracking)),
but here I'm just considering the easiest and the most default case.

On the other hand, Micro ORMs just execute the query, e.g. Dapper Micro ORM:

```csharp
public class PersonDto
{
  public Guid PersonId { get; set; }
  public Guid Name { get; set; }
}

PersonDto GetPerson(Guid wantedPersonId)
{
  return connection.QueryFirst<PersonDto>(
    "select * from Person where PersonId=@PersonId",
    new { PersonId = wantedPersonId });
}
```

and would fetch employees roughly this way:

```csharp
// a reasonable inheritance as an employee IS a person
public class EmployeeDto : PersonDto
{
  public string Position { get; set; }
}

EmployeeDto GetEmployee(Guid wantedPersonId)
{
  return connection.QueryFirst<EmployeeDto>(@"
      select *
      from Employee e join Person p on e.PersonId == p.PersonId
      where PersonId=@PersonId",
    new { PersonId = wantedPersonId });
}
```

IMO, the micro ORM does better, as it does the same thing without having object graph overhead
(both for a machine to execute and for a developer to understand/consider), the syntax is almost as concise.

The ["don't repeat yourself" (DRY)](http://wiki.c2.com/?DontRepeatYourself) principle is also respected
in a sense that PersonDto's properties aren't copied to the EmployeeDto class. Her it looks like a minor detail but trust me --
if you have a database with hundreds of tables (which are sometimes modified) -- maintaining respective DTOs will be error-prone.

There is a tool that should have eased such maintenance -- that is EF's code-first migrations --
but those aren't always possible to apply (e.g. when you have a precious ancient database to work with)
and they are also used reluctantly ("what if I'll not be able to do ... DB change with them").
So I'm considering the case when a database is application independent.
