# Exercise 3: Import the users dataset using Integration Services

> :memo: Extend the provided Integration Services project `integration_services.sln` by modifying the project contents in place.

1. Open the solution with Visual Studio, open _Package.dtsx_ from _Solution explorer_.

   ![Solution explorer of the project](images/exercise/is-solution-explorer.png)

1. Find the _SSIS Toolbox_; you will need to use this window in the following tasks.

1. Add a new _SQL Task_ to the _Control flow_ that erases any content from the target table. You will likely need to run the ETL process multiple times, but the data should only be imported once; hence the initial cleanup task.

   - Drag the _SQL Task_ from the toolbox.
   - Double click to open the details.
   - Specify the connection to the database. Use `ADO NET` connection type, and add a new connection using the dropdown. Use the same connection details as before.
   - Specify a `truncate table <tablename>` [sql statement](https://docs.microsoft.com/en-us/sql/t-sql/statements/truncate-table-transact-sql?view=sql-server-2017) to prune the table contents.

   ![SQL statement task settings](images/exercise/is-create-sql-connection.png)

1. Add a new _Data flow_ element to the control flow pane. Give a meaningful name to this task: "import users". This task should follow the previous cleanup task: draw a line from the previous task box to this one to specify the ordering.

   Your control flow should like like the following at this moment.

   ![Integration Services Control Flow for importing users](images/exercise/is-users-control-flow.png)

1. Open the data flow element by double clicking it.

1. Drag a _Flat file source_ from the toolbox to the data flow pane. Double click to open in. Add a new _Flat file connection manager_ that specifies the source file and its details.

   - Browse the input file from your local disk.
   - Verify that the _Format_ is correct

     - Format: Delimited
     - Text qualifier: specify " (double quotation mark)
     - Check "Column names in the first data row"

       ![Integration Services flat file source settings](images/exercise/is-users-flat-file-conn.png)

   - Switch to the _Columns_ page (within the dialog)
     - Verify that the _Row delimiter_ is `{CR}{LF}` and the _Column delimiter_ is semicolon.
   - Switch to the _Advanced_ page (within the dialog)
     - Verify that the correct number of columns is present
     - Change the UserID to be parsed as `four byte signed integer`
     - Although the age should be a number, it must be parsed as a string here, because there are values in the CSV with the literal `NULL` text
     - Increase the length (_OutputColumnWidth_) of the Location column from 50 to 1000
   - The preview pane is a good way to verify the results. Make sure that there are no lingering quotation marks in the values (or you missed to specify the text qualifier before).
   - And you are done. If you need to modify the settings, you will find this _Connection manager_ at the bottom of the designer surface.

     ![Integration Services connection managers](images/exercise/is-connection-managers.png)

1. In order to split the _Location_ column into two, and to parse the _Age_ as really a number, add a _Derived column_ transformation. Connect the blue output of the file source to into this new box to specify the data flow direction. Then open the settings to specify how the derived values are calculated.

   - Split the location into two. Create a new _City_ and a new _Country_ column using string operations.
     - Take a look at a few examples of the _Location_: they are in the form of "city, state, country"
     - To separate the country find the second comma in the text. The rest is the city (you can keep the state included in the city text). E.g. "nyc, new york, usa" becomes:
       - Country: "usa"
       - City: "nyc, new york"
   - Add a new column that contains the age as an integer. Check if the string is "NULL", in that case keep the null value `NULL(DT_I4)`, or cast to int `(DT_I4)[FieldName]`.
   - Find more information on the syntax of the _Expression_ here: <https://docs.microsoft.com/en-us/sql/integration-services/expressions/integration-services-ssis-expressions?view=sql-server-2017)>

     ![Integration Services Derived column transformation](images/exercise/is-users-derived-col.png)

1. Remove duplicate records. Add a new _Sort_ transformation, connect it to the output of the previous element. Open its settings: chose to sort on the _UserID_, and check the _Remove rows with duplicate sort values_ checkbox.

   ![Integration Services duplicate filtering](images/exercise/is-users-sorting.png)

1. Add an _ADO NET Destination_ component (look in the _Other Destinations_ category) to save the data into database.

   - Direct he output of the previous _Sort_ component here.
   - Open the components settings dialog by double clicking the component box.
   - Create a new _Connection manager_ that connects to the database. Use the same connection settings as before.
   - Select the users table you created as target.

     ![Integration Services ADO NET Destination settings](images/exercise/is-users-adonet-destination.png)

   - Check the mapping and make sure that the right fields go into the right columns. Make sure to double check the _Age_ column because it may exist as a number and as a string too; you need the numbered type here. (Mind, that the column names may be different for you!)

     ![Integration Services ADO NET Destination mapping](images/exercise/is-users-adonet-mapping.png)

1. If you were to execute the process now, it would fail due to errors in the _Derived columns_ transformation. Some content does not correspond to the transformations. Let us skip these rows.

   - Add another _ADO NET Destination_ component.
   - Point the red arrow from the _Derived column_ into this new database destination.

     ![Integration Services Data flow of the Users import](images/exercise/is-users-data-flow.png)

   - A dialog will pop up to specify how to handle errors. For all columns and error types select _Redirect row_. There is a total number of 6 elements to change here. Close this dialog.

     ![Integration Services Error output configuration](images/exercise/is-users-error-output.png)

   - Open the new _ADO NET Destination_ and specify its settings too.

     - Although the result of the transformations are not available, we still save the UserId value. The other database columns will not have any mapping. So effectively only the UserID will be saved to database.

     ![Integration Services database output mapping for transformation errors](images/exercise/is-users-adonet-mapping-for-errors.png)

1. Run the ETL process.

1. If the process succeeded, verify the contents of the table using SQL Server Management Studio. Create a screenshot of the table contents. Please make sure that the screenshot is taken such that it **includes the database name** (which is your Neptun code) from the _Object explorer_ window!

   > :memo: Save the screenshot file as `exercise-3\table_screenshot_users.png` - overwrite the placeholder file with yours.

## Next exercise

Next is [exercise 4](exercise4.md).
