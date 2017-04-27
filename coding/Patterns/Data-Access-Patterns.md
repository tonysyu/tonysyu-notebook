Database Access Patterns
========================

* Row Data Gateway
* Table Data Gateway
* Data Mapper
* Identity Map
* Repository
* etc.


Motivation
----------

- Separation of concerns
    - Reduce cognitive load
        - SQL details or Querying logic should be separate from Domain/Business logic
        - You could have completely different people concerned about DB access and Business Logic
    - Prepare for future changes
        - E.g., switch from SQL server to Oracle
        - E.g., switch from SQL to NoSQL/Object-oriented DB
        - E.g., switch from DB to service-based DB access
- Testing
    - E.g., replace real data-access with in-memory data access

Row Data Gateway
----------------

As the name implies, a Row Data Gateway is an object that acts as a Gateway to
a single row in a database. There are few different variations on the idea, but
the basic idea remains the same

- Database row -> In-memory data

One implementation looks like this:

```csharp
public class UserRowDataGateway
{
    public int UserID { get; set; }
    public string FullName { get; set; }
    public DateTime DOB { get; set; }

    public void Insert(string connection) {...}
    public void Update(string connection) {...}
    public void Delete(string connection) {...}
}
```

- Each property maps to a column in the table
- Handles CRUD operations except for Get/Find
- Let's call this the simple-object variation of Row Data Gateway
  (properties + database-modification operations)

Get and find operations are handled by a separate Finder object

```csharp
public class UserRowDataFinder
{
    public UserRowDataGateway Get(int UserID) {...}
    public List<UserRowDataGateway> FindByLastName(string lastName) {...}
}
```

Usage

```csharp
var userFinder = new UserRowDataFinder(connectionString);
var user = userFinder.Get(userID: 1234);
user.FullName = "Dr.Sniff";
user.Update(connectionString);
```

Row Data Gateway and complex Domain Objects (Entities).

```csharp
// Get Simple Data Objects
var userFinder = new UserRowDataFinder(connectionString);
var user = userFinder.Get(userID: 1234);

var clientFinder = new ClientRowDataFinder(connectionString);
var client = clientFinder.Get(clientCode: "DemoClient");

var userClientFinder = new UserClientRowDataFinder(
var userClient = userClientFinder.Get(userID: user.UserID, clientID: client.ClientID);

// Create Domain Object
var userAccount = new UserClientAccount
{
    user.UserID,
    user.UserName,
    userClient.UserClientID,
    client.ClientID,
    client.ClientCode,
    ...
};
```

### Row Data Gateway Variations

- Simple-object: Properties + Database-modification operations
```csharp
public class UserRowDataGateway_simple
{
    public int UserID { get; set; }
    ...

    public void Insert(string connection) {...}
    ...
}
```
- Complex-object: Properties + Domain Logic + Database-modification operations (Active Record)
```csharp
public class UserRowDataGateway_complex
{
    public int UserID { get; set; }
    ...

    public void Insert(string connection) {...}
    ...

    public void SayHello(int toPersonID) {...}
    ...
}
```
- Access-object: Separate Simple object and Database-modification operations
```csharp
public class UserRowDataGateway_access
{
    public void Insert(string connection, User user) {...}
    ...
}

public class User
{
    public int UserID { get; set; }
    ...
}
```


Table Data Gateway
------------------

An object that acts as a Gateway to a database table. Similar to access-version of RowDataGateway + Finder.

One instance handles all the rows in the table.


```csharp
public class UserTableGateway
{
    public UserTableGateway(string connection) {...}

    public void Insert(User user) {...}
    public void Update(User user) {...}
    public void Delete(int userID) {...}

    public User Get(int userID) {...}
    public List<User> GetAll() {...}
    public List<User> FindByLastName(string lastName) {...}
    ...
}

public class User
{
    public int UserID { get; set; }
    public string FullName { get; set; }
    ...
}
```

Most common variation:

- Instead of passing domain object, pass around primitive values from table


Data Mapper
-----------

A layer of Mappers that moves data between objects and a database
while keeping them independent of each other and the mapper itself.

Sort of _"Man in the Middle"_.

```csharp
class UserClientAccountMapper
{
    public void Insert(UserClientAccount account) {...}
    public void Update(UserClientAccount account) {...}
    public void Delete(int userClientID) {...}
    public void Get(int userClientID) {...}
}

class UserClientAccount
{
    public int UserID { get; set; }
    public string UserName { get; set; }
    public int UserClientID { get; set; }
    public int ClientID { get; set; }
    public string ClientCode { get; set; }
    ...
}
```

- Basically a RowDataGateway (plus get-by-id, but not querying) for objects with data from more than one table.
- Most O/RM are implementations of Data Mapper


Identity Map
------------

Data finder + caching. 
Ensures that each object gets loaded only once by keeping every loaded object in
a map. Looks up objects using the map when referring to them.

```csharp
using AccountStore = Dictionary<int, UserClientAccount>;

class UserClientAccountIdentityMap
{
    private AccountStore _objectStore = new AccountStore();
    private UserClientAccountMapper _objectMapper = new UserClientAccountMapper(connection);

    public UserClientAccount Get(int userClientID)
    {
        if (_objectStore.ContainsKey(userClientID)
        {
            return _objectStore[userClientID];
        }
        else
        {
            var account = _objectMapper.Get(userClientID);
            _objectStore[userClientID] = account;
            return account;
        }
    }
}
```

Especially useful for complex objects that require many table joins.



Repository
----------

- DataMapper for collections.
- Database access + Domain Object (Entity) + Collection interface.
- Dictionary + Database persistence + Bulk operations


Dictionary analogy


```csharp
var userClientDictionary = new Dictionary<int, UserClientAccount>();

// Get
var userClientAccount = userClientDictionary.Get(userClientID);

// Insert or update
userClientDictionary[userClientAccount.ID] = userClientAccount;
```


```csharp
class UserClientAccountRepository
{
    public void Insert(UserClientAccount account) {...}
    public void Insert(List<UserClientAccount> accounts) {...}

    public void Update(UserClientAccount account) {...}
    public void Update(List<UserClientAccount> accounts) {...}

    public void Delete(int userClientID) {...}
    public void Delete(List<int> userClientIDs) {...}

    public UserClientAccount Get(int userClientID) {...}
    public List<UserClientAccount> GetAll() {...}
    public List<UserClientAccount> Find(Func<UserClientAccount, bool> predicate) {...}
}

class UserClientAccount
{
    public int UserID { get; set; }
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public string UserName { get; set; }
    public int UserClientID { get; set; }
    public int ClientID { get; set; }
    public string ClientCode { get; set; }
    ...
}
```

```csharp
var repo = UserClientAccountRepository(connectionString);
var users = repo.Find(account => account.LastName=="Porter");
```

- Domain-Driven Design

Summary of Data Access Patterns
-------------------------------

Here's a _rough_ summary of the core patterns:

|               | Single table data  | Multi-table data |
|---------------|--------------------|------------------|
| Single object | Row Data Gateway   | Data Mapper      |
| Collection    | Table Data Gateway | Repository       |

...

References
----------

- DDD
- Architectural Patterns
- [Martin Fowler's Catalog of Patterns of Enterprise
Application Architecture](http://martinfowler.com/eaaCatalog/index.html)
- Loosely adapted from <https://github.com/willdurand-edu/php-slides/blob/master/src/common/09_databases.md>
