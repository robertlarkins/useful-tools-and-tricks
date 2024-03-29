# Migrations

## Add Migration

`dotnet ef migrations add InitialCreate`

See:
- https://docs.microsoft.com/en-us/ef/core/cli/dotnet


## Change column value

If a column value needs to be changed, such as a nullable column should have empty string values, then the following can be done:

1. Run `Add-Migration`
2. Add the SQL code as a const to the Migration, and as a SQL call:
   ```C#
   public partial class Set_MyTable_MyColumn_Nulls_To_Empty_String : Migration
   {
       private const string SetMyColumnAsEmptyStringWhereNull = @"UPDATE my_table SET my_column = '' WHERE my_column is null;";
       private const string SetMyColumnAsNullWhereEmptyString = @"UPDATE my_table SET my_column = null WHERE my_column = '';";

       protected override void Up(MigrationBuilder migrationBuilder)
       {
           migrationBuilder.Sql(SetMyColumnAsEmptyStringWhereNull);
       }

       protected override void Down(MigrationBuilder migrationBuilder)
       {
           migrationBuilder.Sql(SetMyColumnAsNullWhereEmptyString);
       }
   }
   ```

> Note:
> In the nullable case, ideally the column would be changed to `nullable = false` first.


## Change column type
If a column type needs to be changed there are a couple of options.
The first step is to make modifications to the CodeFirst entities, and run an `Add-Migration`.

If EF Core can't automatically convert the column type, then the better approach is to modify the Migration file to add in custom code for performing the migration.

For example, if the created migration code looks like this:
```C#
public partial class Change_Column_UniqueCode_To_Guid : Migration
{
    protected override void Up(MigrationBuilder migrationBuilder)
    {
        // Either empty, or automatically generated migration code
        // to update the database state from the last migration to this one.
    }

    protected override void Down(MigrationBuilder migrationBuilder)
    {
        // Either empty, or automatically generated migration code
        // to revert the database state from this migration back to the last one.
    }
}
```
and the desire is to convert a string column with guid values in it to a guid column (uuid in postgres), then modify the migration code to look something like this (for a postgres database):
```C#
public partial class Change_Column_UniqueCode_To_Guid : Migration
{
    private const string UniqueCodeStringToGuid = @"ALTER TABLE ""user"" ALTER COLUMN unique_code TYPE uuid USING unique_code::uuid;";
    private const string UniqueCodeGuidToString = @"ALTER TABLE ""user"" ALTER COLUMN unique_code TYPE text USING unique_code::text;";

    protected override void Up(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.Sql(UniqueCodeStringToGuid);
    }

    protected override void Down(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.Sql(UniqueCodeGuidToString);
    }
}
```
In the sql code remember to double quote table and column names that need it, such as if they are uppercase, or are keywords.

Then run `Update-Database` to apply the migration.

Alternatively, you can go to the database and apply the conversion directly, but this is not desirable as
the migration may not be able to automatically run if used in the future due to this manual step.

> Note:
> This approach will only work for the particular type of database (postgres in this case).
> The link to the msdn doc below describes how to accommodate multiple database 

More Info:
 - https://docs.microsoft.com/en-us/ef/core/managing-schemas/migrations/operations


## Custom Mapping of Column to Another Column

If a table `purchase` had a column `purchase_date_time` of type `timestamptz` (which for NodaTime could be OffsetDateTime), and another column `financial_year` of type int was to be added, which is extracted from `purchase_date_time`, then the following steps could be done for EF CodeFirst:


1. Add the new property to the entity, this might be a straight FinancialYear column of type `int`, or it might be wrapped in a ValueType.
   ```C#
   public FinancialYear FinancialYear { get; private set; } = null!;
   ```

2. Add the property to `PurchaseConfiguration`:
   ```C#
   builder.Property(p => p.FinancialYear)
       .HasConversion(p => p.Value, p => FinancialYear.Create(p).Value);
   ```

3. Run `Add-Migration` which will create a migration file similar to this:
   ```C#
    public partial class Add_FinancialYear_Column_To_Purchase : Migration
    {
        protected override void Up(MigrationBuilder migrationBuilder)
        {
            migrationBuilder.AddColumn<int>(
                name: "financial_year",
                table: "purchase",
                nullable: false,
                defaultValue: 0);
        }

        protected override void Down(MigrationBuilder migrationBuilder)
        {
            migrationBuilder.DropColumn(
                name: "financial_year",
                table: "purchase");
        }
    }
   ```
   
4. The next step is to create some SQL to extract the financial year value from the `purchase_date_time` column into the new `financial_year` column, this could look something like this:
   ```C#
    public partial class Add_FinancialYear_Column_To_Purchase : Migration
    {
        private const string SetFinancialYear = @"UPDATE purchase SET financial_year = EXTRACT(year FROM (purchase_date_time + INTERVAL '9 months'));";
    
        protected override void Up(MigrationBuilder migrationBuilder)
        {
            migrationBuilder.AddColumn<int>(
                name: "financial_year",
                table: "purchase",
                nullable: false,
                defaultValue: 0);
                                
            migrationBuilder.Sql(SetFinancialYear);
        }

        protected override void Down(MigrationBuilder migrationBuilder)
        {
            migrationBuilder.DropColumn(
                name: "financial_year",
                table: "purchase");
        }
    }
   ```

5. Run `Update-Database` to apply these changes.


## Map Column from one table to another

If a property `OccurrenceDateTime` on the `Schedule` class is to be shifted to another class, `Appointment`, where `Appointment` has a foreign key to `Schedule`, then running a migration will cause `OccurrenceDateTime` to be dropped from `Schedule` and added to `Appointment`, loosing the data.

```C#
protected override void Up(MigrationBuilder migrationBuilder)
{
   migrationBuilder.DropColumn(
       name: "occurrence_date_time",
       table: "schedule");
       
   migrationBuilder.AddColumn<OffsetDateTime>(
       name: "occurrence_date_time",
       table: "appointment",
       nullable: false,
       defaultValue: new NodaTime.OffsetDateTime(new NodaTime.LocalDateTime(1, 1, 1, 0, 0), NodaTime.Offset.FromHours(0)));
}
```

To fix this, rearrange the two migration code blocks so that the `AddColumn` is before the `DropColumn`. Then add a Sql line to copy the `occurrence_date_time` from `schedule` to the appropriate column in `appointment`. Then add a `migrationBuilder.Sql("some sql")` to do the copying between the two migration pieces:

```C#
private const string CopyOccurrenceDateTimeFromScheduleToAppointment = "UPDATE appointment SET occurrence_date_time = s.occurrence_date_time FROM schedule s WHERE s.id = schedule_id";

protected override void Up(MigrationBuilder migrationBuilder)
{
   migrationBuilder.AddColumn<OffsetDateTime>(
       name: "occurrence_date_time",
       table: "appointment",
       nullable: false,
       defaultValue: new NodaTime.OffsetDateTime(new NodaTime.LocalDateTime(1, 1, 1, 0, 0), NodaTime.Offset.FromHours(0)));

   migrationBuilder.Sql(CopyOccurrenceDateTimeFromScheduleToAppointment);

   migrationBuilder.DropColumn(
       name: "occurrence_date_time",
       table: "schedule");
}
```

There will also be a corresponding `Down` method that will need appropriately updated as well.

## Assign Foreign Key Column a New Value
Sometimes the foreign key value needs to be changed for a migration, for example when a status is to be deleted and needs to be mapped to a new status.
To update the reference, say for the column `appointment_status_id`, then the following examples can be added to the migration to update the database:
```C#
migrationBuilder.Sql("UPDATE appointment " +
                     "SET appointment_status_id = [new_status_id] " +
                     "WHERE appointment_status_id = [old_status_id]");
                     

migrationBuilder.Sql("UPDATE appointment " +
                     "SET appointment_status_id = [new_status_id] " +
                     "WHERE appointment_status_id " +
                     "IN ([old_status_id_1], [old_status_id_2])");
```


## Renaming Migration
If a migration has received an incorrect name, the approach for fixing the name depends on whether there are more recent migrations and if it has gone out to any databases.

### Rename Most Recent Migration

### Rename Existing Migration In the Wild
The worst case is if the migration is _not_ the most recent, and has already been pushed out to client databases.
These steps require access to the __EFMigrationsHistory table on each of the client databases.

Firstly find the migration .cs file that needs renaming, and generally the datetime on the file will be kept as is.
There are five locations that need a new name, so if the migration filename is _20210527103152_wrong_name.cs_:
1. The Migration.cs file name: _20210527103152_The_Right_Name.cs_
2. The Migration file class name: `public partial class TheRightName : Migration`
3. The Designer.cs file name: _20210527103152_The_Right_Name.Designer.cs_
4. The Designer.cs file class name: `partial class TheRightName`
5. The literal migration id in the Designer.cs: `[Migration("20210527103152_The_Right_Name")]`

Then go to each of the client databases and find the MigrationID in the __EFMigrationsHistory table. Tename the MigrationId to the new name.
In the above example this would be _20210527103152_wrong_name_ to _20210527103152_The_Right_Name_.

See: https://stackoverflow.com/questions/44200448/how-to-rename-a-migration-in-entity-framework-core
