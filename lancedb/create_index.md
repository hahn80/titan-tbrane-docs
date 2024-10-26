# Create Index for Vector DB


To create index for a vector database, we can use the following source:

```JAVA
public class CreateIndex {

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
            
            // Step 1: load table
            // Create the nested HashMap for kwargs
            HashMap<String, String> kwargs1 = new HashMap<>();
            kwargs1.put("database_path", "test_db");
            kwargs1.put("table_name", "test");

            // Create the main HashMap for params
            HashMap<String, Object> params1 = new HashMap<>();
            params1.put("task", "loading");
            params1.put("action", "load_table");
            params1.put("kwargs", kwargs1);

            // Step 2: Create the index
            // Create the main HashMap for params2
            HashMap<String, Object> params2 = new HashMap<>();
            params2.put("task", "creating");
            params2.put("action", "create_index");

            // Create the kwargs2 HashMap
            HashMap<String, Object> kwargs2 = new HashMap<>();
            kwargs2.put("table", table_obj);
            kwargs2.put("metric", "L2");
            kwargs2.put("num_partitions", 256);
            kwargs2.put("num_sub_vectors", 96);
            kwargs2.put("accelerator", "cuda"); // or null for CPU
            
            params2.put("kwargs", kwargs2);
            
            table_obj = interpreter.invoke("worker", params2);
            
            System.out.println(table_obj);
        } catch (Exception e) {
            e.printStackTrace(); // Print any exceptions that occur
        }
    }
}
```

The structure of the *params* HashMap:

```JSON
{
  "task": "creating",
  "action": "create_index",
  "kwargs": {
    "table": "<table_obj>",
    "metric": "L2",
    "num_partitions": 256,
    "num_sub_vectors": 96,
    "accelerator": "cuda"
  }
}
```


"task": (String) must be an existing task: `creating`.

"action": (String) must be supported actions. Supported: `create_index`;

"kwargs": another HashMap to pass the arguments to action function: `create_index`.

`metric`: String for the metric. Values: "L2"; "cosine"; "dot"
`num_partitions`: The number of partitions in the index. The default is the square root of the number of rows.
`num_sub_vectors`: The default is the dimension of the vector divided by 16. For example 768/16 = 48
`accelerator`: String; Values: "cuda" for GPU; or null (or ignore this key).

Once we invoked the worker, we now can use *table_obj* for the next computation.

