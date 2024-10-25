# Delete Rows from Tables

To delete rows from table, we can use the following code:

```JAVA
public class DeleteRowsTable {

    public static void main(String[] args) {
        // Create and configure the Python interpreter
        PythonInterpreterConfig config = PythonInterpreterConfig
            .newBuilder()
            .setPythonExec("python") // Specify the Python executable
            //.addPythonPaths(path) // Uncomment and specify if you need to add custom paths
            .build();

        // Use try-with-resources to ensure proper resource management
        try (PythonInterpreter interpreter = new PythonInterpreter(config)) {
            // Import Python worker function
            interpreter.exec("from titan.tbrane.services import worker");
            
            
            // Create the nested HashMap for kwargs
            HashMap<String, String> kwargs = new HashMap<>();
            kwargs.put("database_path", "test_db");
            kwargs.put("table_name", "test");

            // Create the main HashMap for params
            HashMap<String, Object> params = new HashMap<>();
            params.put("task", "loading");
            params.put("action", "load_table");
            params.put("kwargs", kwargs);
            
            
            Object table_obj = interpreter.invoke("worker", params);
            
            HashMap<String, Object> kwargs1 = new HashMap<>();
            kwargs1.put("table", table_obj);
            kwargs1.put("where", "(info = 'IBM')");

            // Create the main HashMap for params
            HashMap<String, Object> params1 = new HashMap<>();
            params1.put("task", "creating");
            params1.put("action", "delete_rows_from_table");
            params1.put("kwargs", kwargs1);

            System.out.println(table_obj);

        } catch (Exception e) {
            e.printStackTrace(); // Print any exceptions that occur
        }
    }
}
```

The structure of the *params* to delete rows:

```JSON
{
  "task": "creating",
  "action": "delete_rows_from_table",
  "kwargs": {
    "table": "<table_obj>",
    "where": "(info = 'IBM')"
  }
}
```

