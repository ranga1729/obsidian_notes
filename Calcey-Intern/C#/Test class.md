#### Arrange
- Create a `IDataContext` object.
- Feed data from the `ReadyMadeTestData` class.
```C#
await SetupTestDataAsync(dataContext);
--------------------------------------
private static async Task SetupTestDataAsync(IDataContext context)
{
    ReadymadeTestData.[table_name].[get_method](context);
    //feed all other data to the context object.
    context.Users.AddRange(ReadymadeTestData.Users.GetUsers());
    context.Customers.AddRange(ReadymadeTestData.Users.GetCustomers());
    context.CustomerNotes.AddRange(GetCustomerNotes());

//save changes to the context obeject.
    await context.SaveChangesAsync();
}
```
- Create mock objects, assign them. Then create a an object of the which includes the method class you want to test. Pass those mock objects to the class object in order to create them. 
```C#
var mockEmailService = new Mock<IEmailService>();
//use the Interfce
var customerNoteService = ServiceHelper.CreateCustomerNoteService(dataContext, mockEmailService.Object);
return customerNoteService;
```
- Create a custom DTO as you want to retrieve the data. 
```C#
var filter = new CustomerNotesSearchFilterDto
{
    PageNumber = pageNumber,
    PageSize = pageSize,
    SortBy = sortBy,
    Ascending = ascending,
    CustomerId = customerId
};
```
#### Act
- Retrieve the data/execute the method
```C#
var records = await service.GetAllCustomerNotesByFilterAsync(filter);
```
#### Assert
```C#
Assert.Equal(totalCount, records.Total);
Assert.Equal(expectedCount, records.Items.Count);
```

****
```C#
public class GetAllCustomerNotesByFilter
{
    [Theory]
    [InlineData(1, 2, "Title", false, "c01", 2, 2)]
    [InlineData(2, 1, "Title", false, "c01", 1, 2)]
    public async Task WhenDataExists_ReturnSuccessfully(int pageNumber, int pageSize, string sortBy,
        bool ascending, string customerId, int expectedCount, int totalCount)
    {

        await DbHelper.ExecuteTestAsync(
         //to setup in memory database
         async (IDataContext dataContext) =>
         {
             //feed data to the in memory database.
             //In memeory db is accessed through the dataContext object.
             await SetupTestDataAsync(dataContext);
         },
         //to set mock objects.
         async (IDataContext dataContext) =>
         {
             //setting a mock IEmail service object.
             var service = CreateService(dataContext);

             //DTO to filter data from the in-memory database
             var filter = new CustomerNotesSearchFilterDto
             {
                 PageNumber = pageNumber,
                 PageSize = pageSize,
                 SortBy = sortBy,
                 Ascending = ascending,
                 CustomerId = customerId
             };

             var records = await service.GetAllCustomerNotesByFilterAsync(filter);

             Assert.Equal(totalCount, records.Total);
             Assert.Equal(expectedCount, records.Items.Count);
         }
         );
    }
}
```