# Optimize Tables


When you add more data to a table, it creates many parts of the input data. Now you can merge them together to get one file on disk and remove all the old one.

The main source to optimize a table

```JAVA
public class OptimizeTable {
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
            HashMap<String, Object> kwargs = new HashMap<>();
            kwargs.put("database_path", "test_db");
            kwargs.put("table_name", "test");
            kwargs.put("older_than", 1);
            kwargs.put("delete_unverified", true);

            // Create the main HashMap for params
            HashMap<String, Object> params = new HashMap<>();
            params.put("task", "loading");
            params.put("action", "optimize_table");
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
  "action": "optimize_table",
  "kwargs": {
    "database_path": "test_db",
    "table_name": "test",
	"older_than": Int (delete all files that is older than number of miliseconds) ,
	"delete_unverified": Bolean (true to delete and false to keep the unverified parts)
    }
  }
}
```


