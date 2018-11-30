# Creating Web API in ASP.NET Core 2.0

## Integration Tests

### Step 08 - Add Integration Tests

In order to add integration tests for API project, follow these steps:

    Right click on Solution > Add > New Project
    Go to Installed > Visual C# > Test > xUnit Test Project (.NET Core)
    Set the name for project as WideWorldImporters.API.IntegrationTests
    Click OK

![Add Integration Tests Project](images/014-AddIntegrationTestsProject.jpg)

Manage references for WideWorldImporters.API.IntegrationTests project:

![Add Reference To Api Project](images/015-AddReferenceToApiProject.jpg)

Now add a reference for WideWorldImporters.API project:

![Reference Manager For Integration Tests Project](images/016-ReferenceManagerForIntegrationTestsProject.jpg)

Once we have created the project, add the following NuGet packages for project:

    Microsoft.AspNetCore.Mvc
    Microsoft.AspNetCore.Mvc.Core
    Microsoft.AspNetCore.Diagnostics
    Microsoft.AspNetCore.TestHost
    Microsoft.Extensions.Configuration.Json

Remove UnitTest1.cs file.

Save changes and build WideWorldImporters.API.IntegrationTests project.

What is the difference between unit tests and integration tests? For unit tests, we simulate all dependencies for Web API project and for integration tests, we run a process that simulates Web API execution, this means Http requests.

Now we proceed to add code related for integration tests.

For this project, integration tests will perform Http requests, each Http request will perform operations to an existing database in SQL Server instance. We'll work with a local instance of SQL Server, this can change according to your working environment, I mean the scope for integration tests.

 Code for TestFixture.cs file

```csharp
using System;
using System.IO;
using System.Net.Http;
using System.Net.Http.Headers;
using System.Reflection;
using Microsoft.AspNetCore.Hosting;
using Microsoft.AspNetCore.Mvc.ApplicationParts;
using Microsoft.AspNetCore.Mvc.Controllers;
using Microsoft.AspNetCore.Mvc.ViewComponents;
using Microsoft.AspNetCore.TestHost;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;

namespace WideWorldImporters.API.IntegrationTests
{
    public class TestFixture<TStartup> : IDisposable
    {
        public static string GetProjectPath(string projectRelativePath, Assembly startupAssembly)
        {
            var projectName = startupAssembly.GetName().Name;

            var applicationBasePath = AppContext.BaseDirectory;

            var directoryInfo = new DirectoryInfo(applicationBasePath);

            do
            {
                directoryInfo = directoryInfo.Parent;

                var projectDirectoryInfo = new DirectoryInfo(Path.Combine(directoryInfo.FullName, projectRelativePath));

                if (projectDirectoryInfo.Exists)
                    if (new FileInfo(Path.Combine(projectDirectoryInfo.FullName, projectName, $"{projectName}.csproj")).Exists)
                        return Path.Combine(projectDirectoryInfo.FullName, projectName);
            }
            while (directoryInfo.Parent != null);

            throw new Exception($"Project root could not be located using the application root {applicationBasePath}.");
        }

        private TestServer Server;

        public TestFixture()
            : this(Path.Combine(""))
        {
        }

        protected TestFixture(string relativeTargetProjectParentDir)
        {
            var startupAssembly = typeof(TStartup).GetTypeInfo().Assembly;
            var contentRoot = GetProjectPath(relativeTargetProjectParentDir, startupAssembly);

            var configurationBuilder = new ConfigurationBuilder()
                .SetBasePath(contentRoot)
                .AddJsonFile("appsettings.json");

            var webHostBuilder = new WebHostBuilder()
                .UseContentRoot(contentRoot)
                .ConfigureServices(InitializeServices)
                .UseConfiguration(configurationBuilder.Build())
                .UseEnvironment("Development")
                .UseStartup(typeof(TStartup));

            Server = new TestServer(webHostBuilder);

            Client = Server.CreateClient();
            Client.BaseAddress = new Uri("http://localhost:1234");
            Client.DefaultRequestHeaders.Accept.Clear();
            Client.DefaultRequestHeaders.Accept.Add(new MediaTypeWithQualityHeaderValue("application/json"));
        }

        public void Dispose()
        {
            Client.Dispose();
            Server.Dispose();
        }

        public HttpClient Client { get; }

        protected virtual void InitializeServices(IServiceCollection services)
        {
            var startupAssembly = typeof(TStartup).GetTypeInfo().Assembly;

            var manager = new ApplicationPartManager();

            manager.ApplicationParts.Add(new AssemblyPart(startupAssembly));
            manager.FeatureProviders.Add(new ControllerFeatureProvider());
            manager.FeatureProviders.Add(new ViewComponentFeatureProvider());

            services.AddSingleton(manager);
        }
    }
}
```

Code for ContentHelper.cs file:

```csharp
using System.Net.Http;
using System.Text;
using Newtonsoft.Json;

namespace WideWorldImporters.API.IntegrationTests
{
    public static class ContentHelper
    {
        public static StringContent GetStringContent(object obj)
            => new StringContent(JsonConvert.SerializeObject(obj), Encoding.Default, "application/json");
    }
}
```

Code for WarehouseTests.cs file:

```csharp
using System;
using System.Net.Http;
using System.Threading.Tasks;
using Newtonsoft.Json;
using WideWorldImporters.API.Models;
using Xunit;

namespace WideWorldImporters.API.IntegrationTests
{
    public class WarehouseTests : IClassFixture<TestFixture<Startup>>
    {
        private HttpClient Client;

        public WarehouseTests(TestFixture<Startup> fixture)
        {
            Client = fixture.Client;
        }

        [Fact]
        public async Task TestGetStockItemsAsync()
        {
            // Arrange
            var request = "/api/v1/Warehouse/StockItem";

            // Act
            var response = await Client.GetAsync(request);

            // Assert
            response.EnsureSuccessStatusCode();
        }

        [Fact]
        public async Task TestGetStockItemAsync()
        {
            // Arrange
            var request = "/api/v1/Warehouse/StockItem/1";

            // Act
            var response = await Client.GetAsync(request);

            // Assert
            response.EnsureSuccessStatusCode();
        }

        [Fact]
        public async Task TestPostStockItemAsync()
        {
            // Arrange
            var request = "/api/v1/Warehouse/StockItem";
            var requestModel = new
            {
                StockItemName = string.Format("USB anime flash drive - Vegeta {0}", Guid.NewGuid()),
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
                SearchDetails = "USB anime flash drive - Vegeta",
                LastEditedBy = 1,
                ValidFrom = DateTime.Now,
                ValidTo = DateTime.Now.AddYears(5)
            };

            // Act
            var response = await Client.PostAsync(request, ContentHelper.GetStringContent(requestModel));
            var value = await response.Content.ReadAsStringAsync();

            // Assert
            response.EnsureSuccessStatusCode();
        }

        [Fact]
        public async Task TestPutStockItemAsync()
        {
            // Arrange
            var requestUrl = "/api/v1/Warehouse/StockItem/1";
            var requestModel = new
            {
                StockItemName = string.Format("USB anime flash drive - Vegeta {0}", Guid.NewGuid()),
                SupplierID = 12,
                Color = 3,
                UnitPrice = 39.00m
            };

            // Act
            var response = await Client.PutAsync(requestUrl, ContentHelper.GetStringContent(requestModel));

            // Assert
            response.EnsureSuccessStatusCode();
        }

        [Fact]
        public async Task TestDeleteStockItemAsync()
        {
            // Arrange
            var postRequest = "/api/v1/Warehouse/StockItem";
            var requestModel = new
            {
                StockItemName = string.Format("Product to delete {0}", Guid.NewGuid()),
                SupplierID = 12,
                UnitPackageID = 7,
                OuterPackageID = 7,
                LeadTimeDays = 14,
                QuantityPerOuter = 1,
                IsChillerStock = false,
                TaxRate = 10.000m,
                UnitPrice = 10.00m,
                RecommendedRetailPrice = 47.84m,
                TypicalWeightPerUnit = 0.050m,
                CustomFields = "{ \"CountryOfManufacture\": \"USA\", \"Tags\": [\"Sample\"] }",
                Tags = "[\"Sample\"]",
                SearchDetails = "Product to delete",
                LastEditedBy = 1,
                ValidFrom = DateTime.Now,
                ValidTo = DateTime.Now.AddYears(5)
            };

            // Act
            var postResponse = await Client.PostAsync(postRequest, ContentHelper.GetStringContent(requestModel));
            var jsonFromPostResponse = await postResponse.Content.ReadAsStringAsync();

            var singleResponse = JsonConvert.DeserializeObject<SingleResponse<StockItem>>(jsonFromPostResponse);

            var deleteResponse = await Client.DeleteAsync(string.Format("/api/v1/Warehouse/StockItem/{0}", singleResponse.Model.StockItemID));

            // Assert
            postResponse.EnsureSuccessStatusCode();

            Assert.False(singleResponse.DidError);

            deleteResponse.EnsureSuccessStatusCode();
        }
    }
}
```

As we can see, WarehouseTests contain all tests for Web API, these are the methods:

|Methods|Description|
|-------|-----------|
|TestGetStockItemsAsync|Retrieves the stock items|
|TestGetStockItemAsync|Retrieves an existing stock item by ID|
|TestPostStockItemAsync|Creates a new stock item|
|TestPutStockItemAsync|Updates an existing stock item|
|TestDeleteStockItemAsync|Deletes an existing stock item|

How Integration Tests Work?

TestFixture<TStartup> class provides a Http client for Web API, uses Startup class from project as reference to apply configurations for client.

WarehouseTests class contains all methods to send Http requests for Web API, the port number for Http client is 1234.

ContentHelper class contains a helper method to create StringContent from request model as JSON, this applies for POST and PUT requests.

The process for integration tests is:

    The Http client in created in class constructor
    Define the request: url and request model (if applies)
    Send the request
    Get the value from response
    Ensure response has success status

Running Integration Tests

Save all changes and build WideWorldImporters.API.IntegrationTests project, test explorer will show all tests in project:

![Test Explorer For Integration Tests](images/017-TestExplorerForIntegrationTests.jpg)

Keep in mind: To execute integration tests, you need to have running an instance of SQL Server, the connection string in appsettings.json file will be used to establish connection with SQL Server.

Now run all integration tests, the test explorer looks like the following image:

![Execution Of Integration Tests](images/018-ExecutionOfIntegrationTests.jpg)

If you get any error executing integration tests, check the error message, review code and repeat the process.

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

* [`Integration tests in ASP.NET Core`](https://docs.microsoft.com/en-us/aspnet/core/test/integration-tests?view=aspnetcore-2.1)
