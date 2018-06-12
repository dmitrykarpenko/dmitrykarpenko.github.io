---
layout: post
title: "Multiple inheritance of DTOs would be a good idea"
date:   2018-05-03 00:00:00 +0000
tags: ORM C-Sharp Concept
comments: true
image:
  src: "/assets/images/arrows-sign.jpg"
  title: "restrict or warn?"
---

## Let's start from afar

C# 8 is to have default interface implementations as one of it's new features
(as it's described e.g. [here](https://stackify.com/csharp-8-features/)).

This feature is described as reasonable and even exciting (as most of the features shipped with every new C# version IMO). The feature reminds good old .h C++ files (with the ability to either have an implementation or not) -- everything new is well-forgotten old indeed.

The feature is also frighteningly reminiscent of another rather arguable C++ concept -- the concept of multiple inheritance and questions like "what implementation will it choose".
It's really a bit disturbing as it leads to one of the following:
* unclearness, ambiguity, and obligation to know more compiler implementation specifics (which could be interesting, but yet redundant);
* cumbersome syntax -- e.g. an obligation to specify a concrete interface to take a method signature (with implementation) from.

Anyway -- time will tell.

And why I'm telling you all this? Because I think there is a reasonable application for multiple inheritance in modern languages (especially back-end ones).

<!--preview-break-->

## Problem

There is a common task, solution of which is still debated -- that is [object-relational mapping](https://en.wikipedia.org/wiki/Object-relational_mapping) and whole [ORM vs Micro ORM debate](http://gunnarpeipman.com/2017/05/micro-orm/).

### ORM

The most popular [Object-relational mapper (ORM)](https://en.wikipedia.org/wiki/Object-relational_mapping) in .NET is [Entity Framework (EF)](https://www.nuget.org/packages/EntityFramework/) (just look at how many downloads it has gotten). Its main advantage is the ability to execute database queries that are written with strongly typed language (e.g. C# LINQ) which leads to much better refactoring, unit testing, and logic versioning. But one of its main disadvantages (especially for beginners striving to write both efficient and clean code) is it's complicated object graphs (especially the detached ones), which vertices are data transfer objects (DTOs) of classes like the following:

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

Sometimes such classes are called entities, POCOs, or models -- depending on a project's naming convention or traditions. [The term DTO was originally related specifically to expensive remote calls](https://martinfowler.com/bliki/LocalDTO.html), I however think it also suits here as database calls are [I/O bound](https://en.wikipedia.org/wiki/I/O_bound) (i.e. somewhat expensive) operation by their nature -- regardless of whether they are remote or not. And data transferring itself is obviously related to databases.

And that's how typical DB queries with Entity Framework look like:

```csharp
Person person = UnitOfWork.Persons
  .FirstOrDefault(p => p.PersonId == wantedPersonId);
// suppose the query is set to join and map the respective person subobject
Employee employee = UnitOfWork.Employees
  .FirstOrDefault(e => e.PersonId == wantedPersonId);

// is true as EF holds a graph of fetched and mapped objects
// and took both references from there
bool doWeHaveAGraph = employee.Person == person;

// one way of updating ...
person.Name = newName;
UnitOfWork.Save();

// ... and the other one that does completely the same
employee.Person.Name = newName;
UnitOfWork.Save();
```

I already mentioned that fetched objects could be detached (i.e. will not be in a tracked graph -- see [tracking](https://docs.microsoft.com/en-us/ef/core/querying/tracking)), but here I'm just considering the easiest and the most default case.

### Micro ORM

On the other hand, Micro ORMs just execute the query, e.g. Dapper Micro ORM:

```csharp
public class PersonDto
{
  public Guid PersonId { get; set; }
  public string Name { get; set; }
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

IMO, here the micro ORM looks better, as it does the same thing without having the object graph overhead (both for a machine to execute and for a developer to understand/consider), and the syntax is almost as concise.

The ["don't repeat yourself" (DRY)](http://wiki.c2.com/?DontRepeatYourself) principle is also respected in a sense that PersonDto's properties aren't copied to the EmployeeDto class. Her it looks like a minor detail but trust me -- if you have a database with hundreds of tables (which are modified sometimes) -- maintaining respective DTOs will be error-prone.

There is a tool that should have eased such maintenance -- that is EF's code-first migrations -- but those aren't always possible to apply (e.g. when you have a valuable legacy database to work with) and they are also used reluctantly (bacause "what if I'll not be able to do ... type of DB change with them"). Therefore, I'm considering the case when a database is application independent.

### Micro ORM and multiple inheritance

So, how multiple inheritance would help us here? Let's suppose we also have a Customer table

```csharp
// a reasonable inheritance as an customer IS also a person
public class CustomerDto : PersonDto
{
  public string CardNumber { get; set; }
}
```

And then you want to get all your employees who also are customers (like those [Apple employees who've used their discount benefit](https://www.quora.com/Do-Apple-employees-get-a-25-discount-on-the-iPhone-6)) -- and the respective DTO should be:

```csharp
public class EmployeeCustomerDto : EmployeeDto, CustomerDto // compilation error
{
  // ...
}
```
This isn't possible now as it's a multiple inheritance -- but this time it's a reasonable one.

On top of that -- such inheritance is not only reasonable in terms of _"is a"_ relation, but also naturally corresponds to the shape of a "joined" query response -- as each row of the query is effectively a flattened set of columns from all the selected tables.

## Idea

Therefore, the idea is to **allow multiple inheritance of DTOs** -- where DTOs would be of classes with only data and no logic -- e.g. they should only contain auto-implemented properties.
The syntax could be something like the following:

```csharp
// WARNING: just a concept, not an actual code!
public dto class FuturePersonDto
{
  public Guid PersonId { get; set; } // OK
  public string Name; // could still be OK

  // not auto-implemented, error
  public string HiddenLogic { get { return PersonId + "_" + Name; } }
}
```

Such improvement would make micro ORM usage even more elegant.
