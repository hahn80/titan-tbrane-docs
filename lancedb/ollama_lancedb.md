# Create Tables Using Ollama


Inititate an empty table using lacedb:

```java
import jcpy.engine.PythonInterpreter;
import jcpy.engine.PythonException;
import jcpy.engine.PythonInterpreterConfig;
import jcpy.engine.object.PyObject;
```

The main source to create an empty table

```JAVA
public class CreateEmptyTable {
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
            HashMap<String, Object> kwargs = new HashMap<>();
            kwargs.put("database_path", "test_db");
            kwargs.put("table_name", "test");
            kwargs.put("source_column", "text");
            kwargs.put("mode", "overwrite");
			
			
			// These settings are for OLLAMA only:			
			kwargs.put("is_ollama", true); // set to true to run ollama
			kwargs.put("model_name_or_path", "bge-m3"); // set model to use with ollama
			kwargs.put("host", "http://localhost:11434"); // set host that runs ollama
			
			Map<String, String> schema = new HashMap<>();

            // Adding entries to the schema HashMap
            schema.put("key", "INT");
            schema.put("text", "STRING");
            schema.put("info", "STRING");
            
            kwargs.put("schema", schema);

            // Create the main HashMap for params
            HashMap<String, Object> params = new HashMap<>();
            params.put("task", "creating");
            params.put("action", "create_empty_table");
            params.put("kwargs", kwargs);
            
            
            Object table_obj = interpreter.invoke("worker", params);
            
            // Now you can get access to the object
            System.out.println(table_obj);
        } catch (Exception e) {
            e.printStackTrace(); // Print any exceptions
        }
    }
}
```

All other functions are the same, but we have to add the followings for ollama.
// These settings are for OLLAMA only:			
kwargs.put("is_ollama", true); // set to true to run ollama
kwargs.put("model_name_or_path", "bge-m3"); // set model to use with ollama
kwargs.put("host", "http://localhost:11434"); // set host that runs ollama
