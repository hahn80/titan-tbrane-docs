# Create Empty Tables


Inititate an empty table using lacedb:

In the preample, we can add some package from the framework:

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
            //kwargs.put("model_name_or_path", "/data/TitanProjects/dev/tbrane/bge-base-en-v1.5");
			
			kwargs.put("model_name_or_path", "M2V_base_output");
			
			// If you load model2vec with static distil model
			// you have to set `is_distill` to be true
			// otherwise you will get error!
            kwargs.put("is_distill", true);
            
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

The structure of the *params* HashMap:

```JSON
{
  "task": "creating",
  "action": "create_empty_table",
  "kwargs": {
    "database_path": "test_db",
    "table_name": "test",
    "source_column": "text",
    "mode": "overwrite",
    "model_name_or_path": "/data/TitanProjects/dev/tbrane/bge-base-en-v1.5",
    "schema": {
      "key": "INT",
      "text": "STRING",
      "info": "STRING"
    }
  }
}
```


"task": (String) must be an existing task: `creating`.

"action": (String) must be supported actions. Supported: `create_empty_table`;

"kwargs": another HashMap to pass the arguments to action function: `create_empty_table`.

Set path to database (String)
kwargs.put("database_path", "/path/to/database");

Set the table name (String)
kwargs.put("table_name", "your_table_name");

Set column name for text field (String)
kwargs.put("source_column", "text");

Set the writing mode for table (String)
kwargs.put("mode", "overwrite"); // The values are *overwrite* or *append*.

Indicate the model name or path (String); support huggingface models.
kwargs.put("model_name_or_path", "/path/to/model");

Set schema (HashMap<String, String>) for the table:
"key": "INT" -> meaning the filed "key" will get the datatype "INT"
"text": "STRING" -> meaing the field "text" will get the datatype "String"
Here is the list of all datatypes:
"BYTE",
"SHORT",
"INT",
"LONG",
"FLOAT",
"DOUBLE",
"BOOLEAN",
"CHAR",
"STRING"


Once we invoked the worker, we now can use *table_obj* for the next computation.

