# Load Tables


If you have a table, we can load it to a table_obj. Make sure you add the preample to run the code.

The main source to load a table

```JAVA
public class LoadTable {
    public static void main(String[] args) {
        // Create and configure the Python interpreter
        PythonInterpreterConfig config = PythonInterpreterConfig
            .newBuilder()
            .setPythonExec("python") // Specify the Python executable
            //.addPythonPaths(path) // Specify if you need to add custom paths
            .build();

        // Use try-with-resources to ensure proper resource management
        try (PythonInterpreter interpreter = new PythonInterpreter(config)) {
            // Import Python worker
            interpreter.exec("from titan.tbrane.services import worker");
            
            // Create the HashMap for kwargs
            HashMap<String, String> kwargs = new HashMap<>();
            kwargs.put("database_path", "test_db");
            kwargs.put("table_name", "test");

            // Create the main HashMap for params
            HashMap<String, Object> params = new HashMap<>();
            params.put("task", "loading");
            params.put("action", "load_table");
            params.put("kwargs", kwargs);
            
            
            Object table_obj = interpreter.invoke("worker", params);
                
            // Now you can use the object
            System.out.println(table_obj);
            

        } catch (Exception e) {
            e.printStackTrace(); // Print any exceptions
        }
    }
}
```

The structure of the *params* HashMap:

```JSON
{
  "task": "loading",
  "action": "load_table",
  "kwargs": {
    "database_path": "test_db",
    "table_name": "test"
    }
  }
}
```


"task": (String) must be an existing task: `loading`.

"action": (String) must be supported actions. Supported: `load_table`;

"kwargs": another HashMap to pass the arguments to action function: `load_table`.

Set path to database (String)
kwargs.put("database_path", "/path/to/database");

Set the table name (String)
kwargs.put("table_name", "your_table_name");

Once we invoked the worker, we now can use *table_obj* for the next computation.

