# Creating Web API in ASP.NET Core 2.0

## Introduction

Let's create a Web API with the latest version of ASP.NET Core and Entity Framework Core.

In this guide, we'll use WideWorldImporters database to create a Web API.

REST APIs provide at least the following operations:

* GET
* POST
* PUT
* DELETE

There are other operations for REST, but they aren't necessary for this guide.

Those operations allow clients to perform actions through REST API, so our Web API must contain those operations.

WideWorldImporters database contains 4 schemas:

Application
Purchasing
Sales
Warehouse

In this guide, we'll work with Warehouse.StockItems table. We'll add code to work with this entity: allow to retrieve stock items, retrieve stock item by id, create, update and delete stock items from database.

The version for this API is 1.

This is the route table for API:

|Verb|Url|Description|
|GET|api/v1/Warehouse/StockItem|Retrieves stock items|
|GET|api/v1/Warehouse/StockItem/id|Retrieves a stock item by id|
|POST|api/v1/Warehouse/StockItem|Creates a new stock item|
|PUT|api/v1/Warehouse/StockItem/id|Updates an existing stock item|
|DELETE|api/v1/Warehouse/StockItem/id|Deletes an existing stock item|

Keep these routes in mind because API must implement all routes.

## Prerequisites

Software

* .NET Core
* NodeJS
* Visual Studio 2017 with last update
* SQL Server
* WideWorldImporters database

Skills

* C#
* ORM (Object Relational Mapping)
* TDD (Test Driven Development)
* RESTful services
    
## Using the Code

For this guide, the working directory for source code is C:\Projects.

### Step 01 - Create Project

Open Visual Studio and follow these steps:

    Go to File > New > Project
    Go to Installed > Visual C# > .NET Core
    Set the name for project as WideWorldImporters.API
    Click OK

In the next window, select API and the latest version for .ASP.NET Core, in this case is 2.1:

Once Visual Studio has finished with creation for solution, we'll see this window:

### Step 02 - Install Nuget Packages

Now, we'll proceed to install EntityFrameworkCore.SqlServer package from Nuget, right click on WideWorldImporters.API project:

Change to Browse tab and type Microsoft.EntityFrameworkCore.SqlServer:

This is the structure for project.

Now run the project to check if solution is ready, press F5 and Visual Studio will show this browser window:

By default, Visual Studio adds a file with name ValuesController in Controllers directory, remove it from project.

### Step 03 - Add Models

Now, create a directory with name Models and add the following files:

    Entities.cs
    Repositories.cs
    Requests.cs
    Responses.cs

Entities.cs will contains all code related to Entity Framework Core.

Repositories.cs will contain the implementation for repositories.

Requests.cs will contain definitions for request models.

Responses.cs will contain definitions for response models.

Code for Entities.cs file:

```csharp
using System;
using Microsoft.EntityFrameworkCore;
using Microsoft.EntityFrameworkCore.Metadata.Builders;

namespace WideWorldImporters.API.Models
{
    public partial class StockItem
    {
        public StockItem()
        {
        }

        public StockItem(int? stockItemID)
        {
            StockItemID = stockItemID;
        }

        public int? StockItemID { get; set; }

        public string StockItemName { get; set; }

        public int? SupplierID { get; set; }

        public int? ColorID { get; set; }

        public int? UnitPackageID { get; set; }

        public int? OuterPackageID { get; set; }

        public string Brand { get; set; }

        public string Size { get; set; }

        public int? LeadTimeDays { get; set; }

        public int? QuantityPerOuter { get; set; }

        public bool? IsChillerStock { get; set; }

        public string Barcode { get; set; }

        public decimal? TaxRate { get; set; }

        public decimal? UnitPrice { get; set; }

        public decimal? RecommendedRetailPrice { get; set; }

        public decimal? TypicalWeightPerUnit { get; set; }

        public string MarketingComments { get; set; }

        public string InternalComments { get; set; }

        public string CustomFields { get; set; }

        public string Tags { get; set; }

        public string SearchDetails { get; set; }

        public int? LastEditedBy { get; set; }

        public DateTime? ValidFrom { get; set; }

        public DateTime? ValidTo { get; set; }
    }

    public class StockItemsConfiguration : IEntityTypeConfiguration<StockItem>
    {
        public void Configure(EntityTypeBuilder<StockItem> builder)
        {
            // Set configuration for entity
            builder.ToTable("StockItems", "Warehouse");

            // Set key for entity
            builder.HasKey(p => p.StockItemID);

            // Set configuration for columns
            builder
                .Property(p => p.StockItemID)
                .HasColumnType("int")
                .IsRequired()
                .HasComputedColumnSql("NEXT VALUE FOR [Sequences].[StockItemID]");

            builder.Property(p => p.StockItemName).HasColumnType("nvarchar(200)").IsRequired();
            builder.Property(p => p.SupplierID).HasColumnType("int").IsRequired();
            builder.Property(p => p.ColorID).HasColumnType("int");
            builder.Property(p => p.UnitPackageID).HasColumnType("int").IsRequired();
            builder.Property(p => p.OuterPackageID).HasColumnType("int").IsRequired();
            builder.Property(p => p.Brand).HasColumnType("nvarchar(100)");
            builder.Property(p => p.Size).HasColumnType("nvarchar(40)");
            builder.Property(p => p.LeadTimeDays).HasColumnType("int").IsRequired();
            builder.Property(p => p.QuantityPerOuter).HasColumnType("int").IsRequired();
            builder.Property(p => p.IsChillerStock).HasColumnType("bit").IsRequired();
            builder.Property(p => p.Barcode).HasColumnType("nvarchar(100)");
            builder.Property(p => p.TaxRate).HasColumnType("decimal(18, 3)").IsRequired();
            builder.Property(p => p.UnitPrice).HasColumnType("decimal(18, 2)").IsRequired();
            builder.Property(p => p.RecommendedRetailPrice).HasColumnType("decimal(18, 2)");
            builder.Property(p => p.TypicalWeightPerUnit).HasColumnType("decimal(18, 3)").IsRequired();
            builder.Property(p => p.MarketingComments).HasColumnType("nvarchar(max)");
            builder.Property(p => p.InternalComments).HasColumnType("nvarchar(max)");
            builder.Property(p => p.CustomFields).HasColumnType("nvarchar(max)");

            builder
                .Property(p => p.Tags)
                .HasColumnType("nvarchar(max)")
                .HasComputedColumnSql("json_query([CustomFields],N'$.Tags')");

            builder
                .Property(p => p.SearchDetails)
                .HasColumnType("nvarchar(max)")
                .IsRequired()
                .HasComputedColumnSql("concat([StockItemName],N' ',[MarketingComments])");

            builder.Property(p => p.LastEditedBy).HasColumnType("int").IsRequired();

            builder
                .Property(p => p.ValidFrom)
                .HasColumnType("datetime2")
                .IsRequired()
                .ValueGeneratedOnAddOrUpdate();

            builder
                .Property(p => p.ValidTo)
                .HasColumnType("datetime2")
                .IsRequired()
                .ValueGeneratedOnAddOrUpdate();
        }
    }

    public class WideWorldImportersDbContext : DbContext
    {
        public WideWorldImportersDbContext(DbContextOptions<WideWorldImportersDbContext> options)
            : base(options)
        {
        }

        protected override void OnModelCreating(ModelBuilder modelBuilder)
        {
            // Apply configurations for entity
            modelBuilder
                .ApplyConfiguration(new StockItemsConfiguration());

            base.OnModelCreating(modelBuilder);
        }

        public DbSet<StockItem> StockItems { get; set; }
    }
}
```

Code for Repositories.cs file:

```csharp
using System;
using System.Linq;
using System.Threading.Tasks;
using Microsoft.EntityFrameworkCore;

namespace WideWorldImporters.API.Models
{
    public interface IRepository : IDisposable
    {
        void Add<TEntity>(TEntity entity) where TEntity : class;

        void Update<TEntity>(TEntity entity) where TEntity : class;

        void Remove<TEntity>(TEntity entity) where TEntity : class;

        int CommitChanges();

        Task<int> CommitChangesAsync();
    }

    public interface IWarehouseRepository : IRepository
    {
        IQueryable<StockItem> GetStockItems(int pageSize = 10, int pageNumber = 1, int? lastEditedBy = null, int? colorID = null, int? outerPackageID = null, int? supplierID = null, int? unitPackageID = null);

        Task<StockItem> GetStockItemsAsync(StockItem entity);

        Task<StockItem> GetStockItemsByStockItemNameAsync(StockItem entity);
    }

    public class Repository
    {
        protected bool Disposed;
        protected WideWorldImportersDbContext DbContext;

        public Repository(WideWorldImportersDbContext dbContext)
        {
            DbContext = dbContext;
        }

        public void Dispose()
        {
            if (!Disposed)
            {
                DbContext?.Dispose();

                Disposed = true;
            }
        }

        public virtual void Add<TEntity>(TEntity entity) where TEntity : class
        {
            // todo: Add code related to addition of entity before to commit changes in database

            DbContext.Add(entity);
        }

        public virtual void Update<TEntity>(TEntity entity) where TEntity : class
        {
            // todo: Add code related to update entity before to commit changes in database

            DbContext.Update(entity);
        }

        public virtual void Remove<TEntity>(TEntity entity) where TEntity : class
        {
            // todo: Add code related to remove entity before to commit changes in database

            DbContext.Remove(entity);
        }

        public int CommitChanges()
            => DbContext.SaveChanges();

        public Task<int> CommitChangesAsync()
            => DbContext.SaveChangesAsync();
    }

    public class WarehouseRepository : Repository, IWarehouseRepository
    {
        public WarehouseRepository(WideWorldImportersDbContext dbContext)
            : base(dbContext)
        {
        }

        public IQueryable<StockItem> GetStockItems(int pageSize = 10, int pageNumber = 1, int? lastEditedBy = null, int? colorID = null, int? outerPackageID = null, int? supplierID = null, int? unitPackageID = null)
        {
            // Get query from DbSet
            var query = DbContext.StockItems.AsQueryable();

            // Filter by: 'LastEditedBy'
            if (lastEditedBy.HasValue)
                query = query.Where(item => item.LastEditedBy == lastEditedBy);

            // Filter by: 'ColorID'
            if (colorID.HasValue)
                query = query.Where(item => item.ColorID == colorID);

            // Filter by: 'OuterPackageID'
            if (outerPackageID.HasValue)
                query = query.Where(item => item.OuterPackageID == outerPackageID);

            // Filter by: 'SupplierID'
            if (supplierID.HasValue)
                query = query.Where(item => item.SupplierID == supplierID);

            // Filter by: 'UnitPackageID'
            if (unitPackageID.HasValue)
                query = query.Where(item => item.UnitPackageID == unitPackageID);

            return query;
        }

        public async Task<StockItem> GetStockItemsAsync(StockItem entity)
            => await DbContext.StockItems.FirstOrDefaultAsync(item => item.StockItemID == entity.StockItemID);

        public async Task<StockItem> GetStockItemsByStockItemNameAsync(StockItem entity)
            => await DbContext.StockItems.FirstOrDefaultAsync(item => item.StockItemName == entity.StockItemName);
    }

    public static class RepositoryExtensions
    {
        public static IQueryable<TEntity> Paging<TEntity>(this WideWorldImportersDbContext dbContext, int pageSize = 0, int pageNumber = 0) where TEntity : class
        {
            var query = dbContext.Set<TEntity>().AsQueryable();

            return pageSize > 0 && pageNumber > 0 ? query.Skip((pageNumber - 1) * pageSize).Take(pageSize) : query;
        }

        public static IQueryable<TModel> Paging<TModel>(this IQueryable<TModel> query, int pageSize = 0, int pageNumber = 0) where TModel : class
            => pageSize > 0 && pageNumber > 0 ? query.Skip((pageNumber - 1) * pageSize).Take(pageSize) : query;
    }
}
```

Code for Requests.cs file:

```csharp
using System;
using System.ComponentModel.DataAnnotations;

namespace WideWorldImporters.API.Models
{
    public class PostStockItemsRequestModel
    {
        [Key]
        public int? StockItemID { get; set; }

        [Required]
        [StringLength(200)]
        public string StockItemName { get; set; }

        [Required]
        public int? SupplierID { get; set; }

        public int? ColorID { get; set; }

        [Required]
        public int? UnitPackageID { get; set; }

        [Required]
        public int? OuterPackageID { get; set; }

        [StringLength(100)]
        public string Brand { get; set; }

        [StringLength(40)]
        public string Size { get; set; }

        [Required]
        public int? LeadTimeDays { get; set; }

        [Required]
        public int? QuantityPerOuter { get; set; }

        [Required]
        public bool? IsChillerStock { get; set; }

        [StringLength(100)]
        public string Barcode { get; set; }

        [Required]
        public decimal? TaxRate { get; set; }

        [Required]
        public decimal? UnitPrice { get; set; }

        public decimal? RecommendedRetailPrice { get; set; }

        [Required]
        public decimal? TypicalWeightPerUnit { get; set; }

        public string MarketingComments { get; set; }

        public string InternalComments { get; set; }

        public string CustomFields { get; set; }

        public string Tags { get; set; }

        [Required]
        public string SearchDetails { get; set; }

        [Required]
        public int? LastEditedBy { get; set; }

        public DateTime? ValidFrom { get; set; }

        public DateTime? ValidTo { get; set; }
    }

    public class PutStockItemsRequestModel
    {
        [Required]
        [StringLength(200)]
        public string StockItemName { get; set; }

        [Required]
        public int? SupplierID { get; set; }

        public int? ColorID { get; set; }

        [Required]
        public decimal? UnitPrice { get; set; }
    }

    public static class Extensions
    {
        public static StockItem ToEntity(this PostStockItemsRequestModel requestModel)
            => new StockItem
            {
                StockItemID = requestModel.StockItemID,
                StockItemName = requestModel.StockItemName,
                SupplierID = requestModel.SupplierID,
                ColorID = requestModel.ColorID,
                UnitPackageID = requestModel.UnitPackageID,
                OuterPackageID = requestModel.OuterPackageID,
                Brand = requestModel.Brand,
                Size = requestModel.Size,
                LeadTimeDays = requestModel.LeadTimeDays,
                QuantityPerOuter = requestModel.QuantityPerOuter,
                IsChillerStock = requestModel.IsChillerStock,
                Barcode = requestModel.Barcode,
                TaxRate = requestModel.TaxRate,
                UnitPrice = requestModel.UnitPrice,
                RecommendedRetailPrice = requestModel.RecommendedRetailPrice,
                TypicalWeightPerUnit = requestModel.TypicalWeightPerUnit,
                MarketingComments = requestModel.MarketingComments,
                InternalComments = requestModel.InternalComments,
                CustomFields = requestModel.CustomFields,
                Tags = requestModel.Tags,
                SearchDetails = requestModel.SearchDetails,
                LastEditedBy = requestModel.LastEditedBy,
                ValidFrom = requestModel.ValidFrom,
                ValidTo = requestModel.ValidTo
            };
    }
}
```

Code for Responses.cs file:

```csharp
using System.Collections.Generic;
using System.Net;
using Microsoft.AspNetCore.Mvc;

namespace WideWorldImporters.API.Models
{
    public interface IResponse
    {
        string Message { get; set; }

        bool DidError { get; set; }

        string ErrorMessage { get; set; }
    }

    public interface ISingleResponse<TModel> : IResponse
    {
        TModel Model { get; set; }
    }

    public interface IListResponse<TModel> : IResponse
    {
        IEnumerable<TModel> Model { get; set; }
    }

    public interface IPagedResponse<TModel> : IListResponse<TModel>
    {
        int ItemsCount { get; set; }

        int PageCount { get; }
    }

    public class SingleResponse<TModel> : ISingleResponse<TModel>
    {
        public string Message { get; set; }

        public bool DidError { get; set; }

        public string ErrorMessage { get; set; }

        public TModel Model { get; set; }
    }

    public class ListResponse<TModel> : IListResponse<TModel>
    {
        public string Message { get; set; }

        public bool DidError { get; set; }

        public string ErrorMessage { get; set; }

        public IEnumerable<TModel> Model { get; set; }
    }

    public class PagedResponse<TModel> : IPagedResponse<TModel>
    {
        public string Message { get; set; }

        public bool DidError { get; set; }

        public string ErrorMessage { get; set; }

        public IEnumerable<TModel> Model { get; set; }

        public int PageSize { get; set; }

        public int PageNumber { get; set; }

        public int ItemsCount { get; set; }

        public int PageCount =>
            PageSize == 0 ? 0 : ItemsCount / PageSize;
    }

    public static class ResponseExtensions
    {
        public static IActionResult ToHttpResponse<TModel>(this IListResponse<TModel> response)
        {
            var status = HttpStatusCode.OK;

            if (response.DidError)
                status = HttpStatusCode.InternalServerError;
            else if (response.Model == null)
                status = HttpStatusCode.NoContent;

            return new ObjectResult(response)
            {
                StatusCode = (int)status
            };
        }

        public static IActionResult ToHttpResponse<TModel>(this ISingleResponse<TModel> response)
        {
            var status = HttpStatusCode.OK;

            if (response.DidError)
                status = HttpStatusCode.InternalServerError;
            else if (response.Model == null)
                status = HttpStatusCode.NotFound;

            return new ObjectResult(response)
            {
                StatusCode = (int)status
            };
        }
    }
}
```

Understanding Models

ENTITIES

StockItems class is the representation for Warehouse.StockItems table.

StockItemsConfiguration class contains the mapping for StockItems class.

WideWorldImportersDbContext class is the link between database and C# code, this class handles queries and commits the changes in database and of course, another things.

REPOSITORIES

IRepository interface represents the model for features.

IWarehouseRepository interface contains all operations related for Warehouse feature, in this guide, we're working only with Warehouse.StockItems table.

Repository class is the abstract model for repositories.

WarehouseRepository contains all implementations for Warehouse feature.

RepositoryExtensions contains extension methods to allow paging in repositories.

REQUESTS

We have the following definitions:

    PostStockItemsRequestModel
    PutStockItemsRequestModel

PostStockItemsRequestModel represents the model to create a new stock item, contains all required properties to save in database.

PutStockItemsRequestModel represents the model to update an existing stock item, in this case contains only 4 properties: StockItemName, SupplierID, ColorID and UnitPrice. This class doesn't contain StockItemID property because id is in route for controller's action.

The models for requests do not require to contain all properties like entities, because we don't need to expose full definition in a request or response, it's a good practice to limit data using models with few properties.

Extensions class contains an extension method for PostStockItemsRequestModel, to return an instance of StockItem class from request model.

RESPONSES

These are the interfaces:

    IResponse
    ISingleResponse<TModel>
    IListResponse<TModel>
    IPagedResponse<TModel>

Each one of these interfaces has implementations, why do we need these definitions if it's more simple to return objects without wrapping them in these models? Keep in mind this Web API will provide operations for clients, with UI or without UI and it's more easy to have properties to send message, to have a model or send information if an error occurs, in addition, we set Http status code in response to describe the result from request.

These classes are generic, because in this way, we save time to define responses in future, this Web API only returns a response for a single entity, a list and a paged list.

ISingleResponse represents a response for a single entity.

IListResponse represents a response with a list, for example all shipping to existing order without paging.

IPagedResponse represents a response with pagination, for example all orders in a date range.

ResponseExtensions class contains extension methods to convert a response in a Http response, these methods return InternalServerError (500) status if an error occurs, OK (200) if it's OK and NotFound (404) if an entity does not exist in database or NoContent (204) for list responses without model.

### Step 04 - Add Controller

### Step 05 - Setting Up Dependency Injection

### Step 06 - Running Web API

### Step 07 - Add Unit Tests

### Step 08 - Add Integration Tests

## Code Challenge

## Code Improvements

## Points of Interest

## History

