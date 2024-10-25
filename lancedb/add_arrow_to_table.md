# Add Arrow Data to Tables


Add arrow data to a table:

```JAVA
public class AddArrowTable {
    public static void main(String[] args) {
        // Configure the Python interpreter
        PythonInterpreterConfig config = PythonInterpreterConfig
            .newBuilder()
            .setPythonExec("python") // Specify the Python executable
            //.addPythonPaths(path) // Specify if you need to add custom paths
            .build();

        // Use try-with-resources to ensure proper resource management
        try (PythonInterpreter interpreter = new PythonInterpreter(config)) {
            // Import Python worker function
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
            
            
            HashMap<String, Object> kwargs1 = new HashMap<>();
            
            kwargs1.put("table", table_obj);
            kwargs1.put("arrow_file", "/path/arrow_file.arrow");
            
            
            
            // Create the main HashMap for params
            HashMap<String, Object> params1 = new HashMap<>();
            params1.put("task", "creating");
            params1.put("action", "add_arrow_to_table");
            params1.put("kwargs", kwargs1);
            
            
            table_obj = interpreter.invoke("worker", params1);
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
  "task": "creating",
  "action": "add_arrow_to_table",
  "kwargs": {
    "table": "<table_obj>",
    "arrow_file": "/path/arrow_file.arrow"
  }
}
```


"task": (String) must be an existing task: `creating`.

"action": (String) must be supported actions. Supported: `add_arrow_to_table`;

"kwargs": another HashMap to pass the arguments to action function: `add_arrow_to_table`.

Set table object (Object)
kwargs.put("table", table object from load_table);

Set json data records
kwargs.put("arrow_file", path to arrow file);

Once we invoked the worker, we now can use *table_obj* for the next computation.

