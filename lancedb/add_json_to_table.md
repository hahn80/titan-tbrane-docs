# Add JSON Data to Tables


Add json data to a table:

```JAVA
public class AddJsonTable {
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
            
            // Create a list to hold HashMap objects
            List<HashMap<String, Object>> records = new ArrayList<>();

            // Create the first HashMap and add entries
            HashMap<String, Object> record1 = new HashMap<>();
            record1.put("key", 10);
            record1.put("text", "A sentence 1");
            record1.put("info", "IBM");

            // Create the second HashMap and add entries
            HashMap<String, Object> record2 = new HashMap<>();
            record2.put("key", 11);
            record2.put("text", "A sentence 2");
            record2.put("info", "MSFT");

            // Add both HashMaps to the list
            records.add(record1);
            records.add(record2);
            
            HashMap<String, Object> kwargs1 = new HashMap<>();
            
            kwargs1.put("table", table_obj);
            kwargs1.put("records", records);
            
            System.out.println(records);
            
            // Create the main HashMap for params
            HashMap<String, Object> params1 = new HashMap<>();
            params1.put("task", "creating");
            params1.put("action", "add_json_to_table");
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
  "action": "add_json_to_table",
  "kwargs": {
    "table": "<table_obj>",
    "records": [
      {
        "key": 10,
        "text": "A sentence 1",
        "info": "IBM"
      },
      {
        "key": 11,
        "text": "A sentence 2",
        "info": "MSFT"
      }
    ]
  }
}

```


"task": (String) must be an existing task: `creating`.

"action": (String) must be supported actions. Supported: `add_json_to_table`;

"kwargs": another HashMap to pass the arguments to action function: `add_json_to_table`.

Set table object (Object)
kwargs.put("table", table object from load_table);

Set json data records
kwargs.put("records", list of HashMaps);

Once we invoked the worker, we now can use *table_obj* for the next computation.

