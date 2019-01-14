# Creating Web API in ASP.NET Core 2.0

## Part 2 - Unit Tests

### Step 07 - Add Unit Tests

In order to add unit tests for API project, follow these steps:

    Right click on Solution > Add > New Project
    Go to Installed > Visual C# > Test > xUnit Test Project (.NET Core)
    Set the name for project as WideWorldImporters.API.UnitTests
    Click OK

![Add Unit Tests Project](images/010-AddUnitTestsProject.jpg)

Manage references for WideWorldImporters.API.UnitTests project:

![Add Reference To Api Project](images/011-AddReferenceToApiProject.jpg)

Now add a reference for WideWorldImporters.API project:

![Reference Manager for Unit Tests Project](images/012-ReferenceManagerForUnitTestsProject.jpg)

Once we have created the project, add the following NuGet packages for project:

    Microsoft.AspNetCore.Mvc.Core
    Microsoft.EntityFrameworkCore.InMemory

Remove UnitTest1.cs file.

Save changes and build WideWorldImporters.API.UnitTests project.

Now we proceed to add code related for unit tests, these tests will work with In memory database.

What is TDD? Testing is a common practice in nowadays, because with unit tests, it's easy to performing tests for features before to publish, Test Driven Development (TDD) is the way to define unit tests and validate the behavior in code.

Another concept in TDD is AAA: Arrange, Act and Assert; Arrange is the block for creation of objects, Act is the block to place all invocations for methods and Assert is the block to validate the results from methods invocation.

Since We're working with In memory database for unit tests, We need to create a class to mock WideWorldImportersDbContext class and also add data to perform testing for WideWorldImportersDbContext extension methods.

To be clear: these unit tests do not establish a connection with SQL Server.

For unit tests, add the following files:

    DbContextExtensions.cs
    DbContextMocker.cs
    WarehouseControllerUnitTest.cs

Code for DbContextExtensions.cs file:

```csharp
using System;
using WideWorldImporters.API.Models;

namespace WideWorldImporters.API.UnitTests
{
    public static class DbContextExtensions
    {
        public static void Seed(this WideWorldImportersDbContext dbContext)
        {
            // Add entities for DbContext instance

            dbContext.StockItems.Add(new StockItem
            {
                StockItemID = 1,
                StockItemName = "USB missile launcher (Green)",
                SupplierID = 12,
                UnitPackageID = 7,
                OuterPackageID = 7,
                LeadTimeDays = 14,
                QuantityPerOuter = 1,
                IsChillerStock = false,
                TaxRate = 15.000m,
                UnitPrice = 25.00m,
                RecommendedRetailPrice = 37.38m,
                TypicalWeightPerUnit = 0.300m,
                MarketingComments = "Complete with 12 projectiles",
                CustomFields = "{ \"CountryOfManufacture\": \"China\", \"Tags\": [\"USB Powered\"] }",
                Tags = "[\"USB Powered\"]",
                SearchDetails = "USB missile launcher (Green) Complete with 12 projectiles",
                LastEditedBy = 1,
                ValidFrom = Convert.ToDateTime("5/31/2016 11:11:00 PM"),
                ValidTo = Convert.ToDateTime("12/31/9999 11:59:59 PM")
            });

            dbContext.StockItems.Add(new StockItem
            {
                StockItemID = 2,
                StockItemName = "USB rocket launcher (Gray)",
                SupplierID = 12,
                ColorID = 12,
                UnitPackageID = 7,
                OuterPackageID = 7,
                LeadTimeDays = 14,
                QuantityPerOuter = 1,
                IsChillerStock = false,
                TaxRate = 15.000m,
                UnitPrice = 25.00m,
                RecommendedRetailPrice = 37.38m,
                TypicalWeightPerUnit = 0.300m,
                MarketingComments = "Complete with 12 projectiles",
                CustomFields = "{ \"CountryOfManufacture\": \"China\", \"Tags\": [\"USB Powered\"] }",
                Tags = "[\"USB Powered\"]",
                SearchDetails = "USB rocket launcher (Gray) Complete with 12 projectiles",
                LastEditedBy = 1,
                ValidFrom = Convert.ToDateTime("5/31/2016 11:11:00 PM"),
                ValidTo = Convert.ToDateTime("12/31/9999 11:59:59 PM")
            });

            dbContext.StockItems.Add(new StockItem
            {
                StockItemID = 3,
                StockItemName = "Office cube periscope (Black)",
                SupplierID = 12,
                ColorID = 3,
                UnitPackageID = 7,
                OuterPackageID = 6,
                LeadTimeDays = 14,
                QuantityPerOuter = 10,
                IsChillerStock = false,
                TaxRate = 15.000m,
                UnitPrice = 18.50m,
                RecommendedRetailPrice = 27.66m,
                TypicalWeightPerUnit = 0.250m,
                MarketingComments = "Need to see over your cubicle wall? This is just what's needed.",
                CustomFields = "{ \"CountryOfManufacture\": \"China\", \"Tags\": [] }",
                Tags = "[]",
                SearchDetails = "Office cube periscope (Black) Need to see over your cubicle wall? This is just what's needed.",
                LastEditedBy = 1,
                ValidFrom = Convert.ToDateTime("5/31/2016 11:00:00 PM"),
                ValidTo = Convert.ToDateTime("12/31/9999 11:59:59 PM")
            });

            dbContext.StockItems.Add(new StockItem
            {
                StockItemID = 4,
                StockItemName = "USB food flash drive - sushi roll",
                SupplierID = 12,
                UnitPackageID = 7,
                OuterPackageID = 7,
                LeadTimeDays = 14,
                QuantityPerOuter = 1,
                IsChillerStock = false,
                TaxRate = 15.000m,
                UnitPrice = 32.00m,
                RecommendedRetailPrice = 47.84m,
                TypicalWeightPerUnit = 0.050m,
                CustomFields = "{ \"CountryOfManufacture\": \"Japan\", \"Tags\": [\"32GB\",\"USB Powered\"] }",
                Tags = "[\"32GB\",\"USB Powered\"]",
                SearchDetails = "USB food flash drive - sushi roll ",
                LastEditedBy = 1,
                ValidFrom = Convert.ToDateTime("5/31/2016 11:11:00 PM"),
                ValidTo = Convert.ToDateTime("12/31/9999 11:59:59 PM")
            });

            dbContext.StockItems.Add(new StockItem
            {
                StockItemID = 5,
                StockItemName = "USB food flash drive - hamburger",
                SupplierID = 12,
                UnitPackageID = 7,
                OuterPackageID = 7,
                LeadTimeDays = 14,
                QuantityPerOuter = 1,
                IsChillerStock = false,
                TaxRate = 15.000m,
                UnitPrice = 32.00m,
                RecommendedRetailPrice = 47.84m,
                TypicalWeightPerUnit = 0.050m,
                CustomFields = "{ \"CountryOfManufacture\": \"Japan\", \"Tags\": [\"16GB\",\"USB Powered\"] }",
                Tags = "[\"16GB\",\"USB Powered\"]",
                SearchDetails = "USB food flash drive - hamburger ",
                LastEditedBy = 1,
                ValidFrom = Convert.ToDateTime("5/31/2016 11:11:00 PM"),
                ValidTo = Convert.ToDateTime("12/31/9999 11:59:59 PM")
            });

            dbContext.StockItems.Add(new StockItem
            {
                StockItemID = 6,
                StockItemName = "USB food flash drive - hot dog",
                SupplierID = 12,
                UnitPackageID = 7,
                OuterPackageID = 7,
                LeadTimeDays = 14,
                QuantityPerOuter = 1,
                IsChillerStock = false,
                TaxRate = 15.000m,
                UnitPrice = 32.00m,
                RecommendedRetailPrice = 47.84m,
                TypicalWeightPerUnit = 0.050m,
                CustomFields = "{ \"CountryOfManufacture\": \"Japan\", \"Tags\": [\"32GB\",\"USB Powered\"] }",
                Tags = "[\"32GB\",\"USB Powered\"]",
                SearchDetails = "USB food flash drive - hot dog ",
                LastEditedBy = 1,
                ValidFrom = Convert.ToDateTime("5/31/2016 11:11:00 PM"),
                ValidTo = Convert.ToDateTime("12/31/9999 11:59:59 PM")
            });

            dbContext.StockItems.Add(new StockItem
            {
                StockItemID = 7,
                StockItemName = "USB food flash drive - pizza slice",
                SupplierID = 12,
                UnitPackageID = 7,
                OuterPackageID = 7,
                LeadTimeDays = 14,
                QuantityPerOuter = 1,
                IsChillerStock = false,
                TaxRate = 15.000m,
                UnitPrice = 32.00m,
                RecommendedRetailPrice = 47.84m,
                TypicalWeightPerUnit = 0.050m,
                CustomFields = "{ \"CountryOfManufacture\": \"Japan\", \"Tags\": [\"16GB\",\"USB Powered\"] }",
                Tags = "[\"16GB\",\"USB Powered\"]",
                SearchDetails = "USB food flash drive - pizza slice ",
                LastEditedBy = 1,
                ValidFrom = Convert.ToDateTime("5/31/2016 11:11:00 PM"),
                ValidTo = Convert.ToDateTime("12/31/9999 11:59:59 PM")
            });

            dbContext.StockItems.Add(new StockItem
            {
                StockItemID = 8,
                StockItemName = "USB food flash drive - dim sum 10 drive variety pack",
                SupplierID = 12,
                UnitPackageID = 9,
                OuterPackageID = 9,
                LeadTimeDays = 14,
                QuantityPerOuter = 1,
                IsChillerStock = false,
                TaxRate = 15.000m,
                UnitPrice = 240.00m,
                RecommendedRetailPrice = 358.80m,
                TypicalWeightPerUnit = 0.500m,
                CustomFields = "{ \"CountryOfManufacture\": \"Japan\", \"Tags\": [\"32GB\",\"USB Powered\"] }",
                Tags = "[\"32GB\",\"USB Powered\"]",
                SearchDetails = "USB food flash drive - dim sum 10 drive variety pack ",
                LastEditedBy = 1,
                ValidFrom = Convert.ToDateTime("5/31/2016 11:11:00 PM"),
                ValidTo = Convert.ToDateTime("12/31/9999 11:59:59 PM")
            });

            dbContext.StockItems.Add(new StockItem
            {
                StockItemID = 9,
                StockItemName = "USB food flash drive - banana",
                SupplierID = 12,
                UnitPackageID = 7,
                OuterPackageID = 7,
                LeadTimeDays = 14,
                QuantityPerOuter = 1,
                IsChillerStock = false,
                TaxRate = 15.000m,
                UnitPrice = 32.00m,
                RecommendedRetailPrice = 47.84m,
                TypicalWeightPerUnit = 0.050m,
                CustomFields = "{ \"CountryOfManufacture\": \"Japan\", \"Tags\": [\"16GB\",\"USB Powered\"] }",
                Tags = "[\"16GB\",\"USB Powered\"]",
                SearchDetails = "USB food flash drive - banana ",
                LastEditedBy = 1,
                ValidFrom = Convert.ToDateTime("5/31/2016 11:11:00 PM"),
                ValidTo = Convert.ToDateTime("12/31/9999 11:59:59 PM")
            });

            dbContext.StockItems.Add(new StockItem
            {
                StockItemID = 10,
                StockItemName = "USB food flash drive - chocolate bar",
                SupplierID = 12,
                UnitPackageID = 7,
                OuterPackageID = 7,
                LeadTimeDays = 14,
                QuantityPerOuter = 1,
                IsChillerStock = false,
                TaxRate = 15.000m,
                UnitPrice = 32.00m,
                RecommendedRetailPrice = 47.84m,
                TypicalWeightPerUnit = 0.050m,
                CustomFields = "{ \"CountryOfManufacture\": \"Japan\", \"Tags\": [\"32GB\",\"USB Powered\"] }",
                Tags = "[\"32GB\",\"USB Powered\"]",
                SearchDetails = "USB food flash drive - chocolate bar ",
                LastEditedBy = 1,
                ValidFrom = Convert.ToDateTime("5/31/2016 11:11:00 PM"),
                ValidTo = Convert.ToDateTime("12/31/9999 11:59:59 PM")
            });

            dbContext.StockItems.Add(new StockItem
            {
                StockItemID = 11,
                StockItemName = "USB food flash drive - cookie",
                SupplierID = 12,
                UnitPackageID = 7,
                OuterPackageID = 7,
                LeadTimeDays = 14,
                QuantityPerOuter = 1,
                IsChillerStock = false,
                TaxRate = 15.000m,
                UnitPrice = 32.00m,
                RecommendedRetailPrice = 47.84m,
                TypicalWeightPerUnit = 0.050m,
                CustomFields = "{ \"CountryOfManufacture\": \"Japan\", \"Tags\": [\"16GB\",\"USB Powered\"] }",
                Tags = "[\"16GB\",\"USB Powered\"]",
                SearchDetails = "USB food flash drive - cookie ",
                LastEditedBy = 1,
                ValidFrom = Convert.ToDateTime("5/31/2016 11:11:00 PM"),
                ValidTo = Convert.ToDateTime("12/31/9999 11:59:59 PM")
            });

            dbContext.StockItems.Add(new StockItem
            {
                StockItemID = 12,
                StockItemName = "USB food flash drive - donut",
                SupplierID = 12,
                UnitPackageID = 7,
                OuterPackageID = 7,
                LeadTimeDays = 14,
                QuantityPerOuter = 1,
                IsChillerStock = false,
                TaxRate = 15.000m,
                UnitPrice = 32.00m,
                RecommendedRetailPrice = 47.84m,
                TypicalWeightPerUnit = 0.050m,
                CustomFields = "{ \"CountryOfManufacture\": \"Japan\", \"Tags\": [\"32GB\",\"USB Powered\"] }",
                Tags = "[\"32GB\",\"USB Powered\"]",
                SearchDetails = "USB food flash drive - donut ",
                LastEditedBy = 1,
                ValidFrom = Convert.ToDateTime("5/31/2016 11:11:00 PM"),
                ValidTo = Convert.ToDateTime("12/31/9999 11:59:59 PM")
            });

            dbContext.SaveChanges();
        }
    }
}
```

Code for DbContextMocker.cs file:

```csharp
using Microsoft.EntityFrameworkCore;
using WideWorldImporters.API.Models;

namespace WideWorldImporters.API.UnitTests
{
    public static class DbContextMocker
    {
        public static WideWorldImportersDbContext GetWideWorldImportersDbContext(string dbName)
        {
            // Create options for DbContext instance
            var options = new DbContextOptionsBuilder<WideWorldImportersDbContext>()
                .UseInMemoryDatabase(databaseName: dbName)
                .Options;

            // Create instance of DbContext
            var dbContext = new WideWorldImportersDbContext(options);

            // Add entities in memory
            dbContext.Seed();

            return dbContext;
        }
    }
}
```

Code for WarehouseControllerUnitTest.cs file:

```csharp
using System;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Mvc;
using WideWorldImporters.API.Controllers;
using WideWorldImporters.API.Models;
using Xunit;

namespace WideWorldImporters.API.UnitTests
{
    public class WarehouseControllerUnitTest
    {
        [Fact]
        public async Task TestGetStockItemsAsync()
        {
            // Arrange
            var dbContext = DbContextMocker.GetWideWorldImportersDbContext(nameof(TestGetStockItemsAsync));
            var controller = new WarehouseController(null, dbContext);

            // Act
            var response = await controller.GetStockItemsAsync() as ObjectResult;
            var value = response.Value as IPagedResponse<StockItem>;

            dbContext.Dispose();

            // Assert
            Assert.False(value.DidError);
        }

        [Fact]
        public async Task TestGetStockItemAsync()
        {
            // Arrange
            var dbContext = DbContextMocker.GetWideWorldImportersDbContext(nameof(TestGetStockItemAsync));
            var controller = new WarehouseController(null, dbContext);
            var id = 1;

            // Act
            var response = await controller.GetStockItemAsync(id) as ObjectResult;
            var value = response.Value as ISingleResponse<StockItem>;

            dbContext.Dispose();

            // Assert
            Assert.False(value.DidError);
        }

        [Fact]
        public async Task TestPostStockItemAsync()
        {
            // Arrange
            var dbContext = DbContextMocker.GetWideWorldImportersDbContext(nameof(TestPostStockItemAsync));
            var controller = new WarehouseController(null, dbContext);
            var request = new PostStockItemsRequest
            {
                StockItemID = 100,
                StockItemName = "USB anime flash drive - Goku",
                SupplierID = 12,
                UnitPackageID = 7,
                OuterPackageID = 7,
                LeadTimeDays = 14,
                QuantityPerOuter = 1,
                IsChillerStock = false,
                TaxRate = 15.000m,
                UnitPrice = 32.00m,
                RecommendedRetailPrice = 47.84m,
                TypicalWeightPerUnit = 0.050m,
                CustomFields = "{ \"CountryOfManufacture\": \"Japan\", \"Tags\": [\"32GB\",\"USB Powered\"] }",
                Tags = "[\"32GB\",\"USB Powered\"]",
                SearchDetails = "USB anime flash drive - Goku",
                LastEditedBy = 1,
                ValidFrom = DateTime.Now,
                ValidTo = DateTime.Now.AddYears(5)
            };

            // Act
            var response = await controller.PostStockItemAsync(request) as ObjectResult;
            var value = response.Value as ISingleResponse<StockItem>;

            dbContext.Dispose();

            // Assert
            Assert.False(value.DidError);
        }

        [Fact]
        public async Task TestPutStockItemAsync()
        {
            // Arrange
            var dbContext = DbContextMocker.GetWideWorldImportersDbContext(nameof(TestPutStockItemAsync));
            var controller = new WarehouseController(null, dbContext);
            var id = 12;
            var request = new PutStockItemsRequest
            {
                StockItemName = "USB food flash drive (Update)",
                SupplierID = 12,
                ColorID = 3
            };

            // Act
            var response = await controller.PutStockItemAsync(id, request) as ObjectResult;
            var value = response.Value as IResponse;

            dbContext.Dispose();

            // Assert
            Assert.False(value.DidError);
        }

        [Fact]
        public async Task TestDeleteStockItemAsync()
        {
            // Arrange
            var dbContext = DbContextMocker.GetWideWorldImportersDbContext(nameof(TestDeleteStockItemAsync));
            var controller = new WarehouseController(null, dbContext);
            var id = 5;

            // Act
            var response = await controller.DeleteStockItemAsync(id) as ObjectResult;
            var value = response.Value as IResponse;

            dbContext.Dispose();

            // Assert
            Assert.False(value.DidError);
        }
    }
}
```

As We can see, WarehouseControllerUnitTest contains all tests for Web API, these are the methods:

|Method|Description|
|------|-----------|
|TestGetStockItemsAsync|Retrieves the stock items|
|TestGetStockItemAsync|Retrieves an existing stock item by ID|
|TestPostStockItemAsync|Creates a new stock item|
|TestPutStockItemAsync|Updates an existing stock item|
|TestDeleteStockItemAsync|Deletes an existing stock item|

How Unit Tests Work?

DbContextMocker creates an instance of WideWorldImportersDbContext using in memory database, the dbName parameter sets the name for in memory database; then there is an invocation for Seed method, this method adds entities for WideWorldImportersDbContext instance in order to provide results.

DbContextExtensions class contains Seed extension method.

WarehouseControllerUnitTest class contains all tests for WarehouseController class.

Keep in mind each test uses a different in memory database, inside of each test method. We retrieve in memory database using the name of test method with nameof operator.

At this level (Unit tests), we only need to check the operations for repositories, there is no need to work with a SQL database (relations, transactions, etc.).

The process for unit tests is:

    Create an instance of DbContext
    Create an instance of controller
    Invoke controller's method
    Get value from controller's invocation
    Dispose DbContext instance
    Validate response

Running Unit Tests

Save all changes and build WideWorldImporters.API.UnitTests project.

Now, check tests in test explorer:

![Test Explorer For Unit Tests](images/013-TestExplorerForUnitTests.jpg)

Run all tests using test explorer, if you get any error, check the error message, review code and repeat the process.

## Code Challenge

At this point, you have skills to extend API, take this as a challenge for you and add the following tests:

|Test|Description|
|----|-----------|
|Get stock items by parameters|Make a request for stock items searching by lastEditedBy, colorID, outerPackageID, supplierID, unitPackageID parameters.|
|Get a non existing stock item|Get a stock item using a non existing ID and check Web API returns NotFound (404) status.|
|Add a stock item with existing name|Add a stock item with an existing name and check Web API returns BadRequest (400) status.|
|Add a stock item without required fields|Add a stock item without required fields and check Web API returns BadRequest (400) status.|
|Update a non existing stock item|Update a stock item using a non existing ID and check Web API returns NotFound (404) status.|
|Update an existing stock item without required fields|Update an existing stock item without required fields and check Web API returns BadRequest (400) status.|
|Delete a non existing stock item|Delete a stock item using a non existing ID and check Web API returns NotFound (404) status.|
|Delete a stock item with orders|Delete a stock item using a non existing ID and check Web API returns NotFound (404) status.|

Follow the convention used in unit and integration tests to complete this challenge.

Good luck!

### Related Links

* [`Unit testing C# in .NET Core using dotnet test and xUnit`](https://docs.microsoft.com/en-us/dotnet/core/testing/unit-testing-with-dotnet-test?view=aspnetcore-2.1)
