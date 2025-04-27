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
            interpreter.exec("from titan.tbrane.services import tbrane_worker");
            
            // Create the HashMap for kwargs
            HashMap<String, Object> kwargs = new HashMap<>();
            kwargs.put("database_path", "test_db");
            kwargs.put("table_name", "test");
			kwargs.put("arrow_file", "/path/arrow_file.arrow");
			kwargs.put("mode", "append");

            
            // Create the main HashMap for params
            HashMap<String, Object> params = new HashMap<>();
            params.put("task", "creating");
            params.put("action", "add_arrow_to_table");
            params.put("kwargs", kwargs);
            
            
            Object table_obj = interpreter.invoke("worker", params);
            System.out.println(table_obj);

        } catch (Exception e) {
            e.printStackTrace(); // Print any exceptions
        }
    }
}
```

Notice: Make sure your arrow file has the same schema as database table schema.


