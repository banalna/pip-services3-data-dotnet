# <img src="https://uploads-ssl.webflow.com/5ea5d3315186cf5ec60c3ee4/5edf1c94ce4c859f2b188094_logo.svg" alt="Pip.Services Logo" width="200"> <br/> Persistence components for .NET

This module is a part of the [Pip.Services](http://pipservices.org) polyglot microservices toolkit. It contains generic interfaces for data access components as well as abstract implementations for in-memory and file persistence.

The persistence components come in two kinds. The first kind is a basic persistence that can work with any object types and provides only minimal set of operations. 
The second kind is so called "identifieable" persistence with works with "identifable" data objects, i.e. objects that have unique ID field. The identifiable persistence provides a full set or CRUD operations that covers most common cases.

The module contains the following packages:
- **Core** - generic interfaces for data access components. 
- **Persistence** - in-memory and file persistence components, as well as JSON persister class.

<a name="links"></a> Quick links:

* [Memory persistence](https://www.pipservices.org/recipies/memory-persistence)
* [API Reference](https://pip-services3-dotnet.github.io/pip-services3-data-dotnet)
* [Change Log](CHANGELOG.md)
* [Get Help](https://www.pipservices.org/community/help)
* [Contribute](https://www.pipservices.org/community/contribute)

## Use

Install the dotnet package as
```bash
dotnet add package PipServices3.Data
```

For example, you need to implement persistence for a data object defined as following.

```cs
using PipServices3.Commons.Data;

class MyObject : IIdentifiable<string>
{
    string IIdentifiable<string>.Id { get => this.Id; set => this.Id = value; }

    private string Id;
    public string key;
    public int value;
}
```

Our persistence component shall implement the following interface with a basic set of CRUD operations.

```cs
interface IMyPersistance
{
    void GetPageByFilter(string correlationId, FilterParams filter, PagingParams paging);

    void GetOneById(string correlationId, string id);

    void GetOneByKey(string correlationId, string id);

    void Create(string correlationId, MyObject item);

    void Update(string correlationId, MyObject item);

    void DeleteById(string correlationId, string id);
}
```

To implement in-memory persistence component you shall inherit `IdentifiableMemoryPersistence`. 
Most CRUD operations will come from the base class. You only need to override `GetPageByFilter` method with a custom filter function.
And implement a `GetOneByKey` custom persistence method that doesn't exist in the base class.

```cs
using PipServices3.Data.Persistence;

class MyMemoryPersistence : IdentifiableMemoryPersistence<MyObject, string>
{
    public MyMemoryPersistence() : base() { }

    private List<Func<MyData, bool>> ComposeFilter(FilterParams filter)
    {
        filter = filter != null ? filter : new FilterParams();

        string id = filter.GetAsNullableString("id");
        string tempIds = filter.GetAsNullableString("ids");
        string[] ids = tempIds != null ? tempIds.Split(",") : null;
        string key = filter.GetAsNullableString("key");

        return new List<Func<MyData, bool>>() {
            (item) => {
                if (name != null && item.name != name)
                    return false;
                return true;
            }
        };
    }

    void GetPageByFilter(string correlationId, FilterParams filter, PagingParams paging)
    {
        base.GetPageByFilterAsync(correlationId, this.ComposeFilter(filter), paging);
    }

    void GetOneByKey(string correlationId, List<MyObject> item, string key)
    {
        List<MyObject> items = item.Find(x => x.key == key);

        if (item.Count > 0)
            this._logger.trace(correlationId, "Found object by key=%s", key);
        else
            this._logger.trace(correlationId, "Cannot find by key=%s", key);
    }

}
```

It is easy to create file persistence by adding a persister object to the implemented in-memory persistence component.

```cs
using PipServices3.Commons.Config;
using PipServices3.Data.Persistence;

class MyFilePersistence : MyMemoryPersistence
{
    protected JsonFilePersister<MyObject> _persister;

    MyFilePersistence(string path = null) : base()
    {
        this._persister = new JsonFilePersister<MyObject>(path);
        this._loader = this._persister;
        this._saver = this._persister;
    }

    public void Configure(ConfigParams config)
    {
        base.Configure(config);
        this._persister.Configure(config);
    }
}
```

## Develop

For development you shall install the following prerequisites:
* Core .NET SDK 3.1+
* Visual Studio Code or another IDE of your choice
* Docker

Restore dependencies:
```bash
dotnet restore src/src.csproj
```

Compile the code:
```bash
dotnet build src/src.csproj
```

Run automated tests:
```bash
dotnet restore test/test.csproj
dotnet test test/test.csproj
```

Generate API documentation:
```bash
./docgen.ps1
```

Before committing changes run dockerized build and test as:
```bash
./build.ps1
./test.ps1
./clear.ps1
```

## Contacts

The .NET version of Pip.Services is created and maintained by:
- **Volodymyr Tkachenko**
- **Sergey Seroukhov**
- **Mark Zontak**
- **Alex Mazur**