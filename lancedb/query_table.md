# Query Vector DB

Once the vector database is created, we now can query the database to get the result for RAG model.

```JAVA
public class QueryTable {

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
            
            // Step 2: query the table
            // Create the main HashMap for params2
            HashMap<String, Object> params2 = new HashMap<>();
            params2.put("task", "querying");
            params2.put("action", "query_from_table");
            
            
            // Create the kwargs2 HashMap
            HashMap<String, Object> kwargs2 = new HashMap<>();
            kwargs2.put("limit", 3);
            
            params2.put("kwargs", kwargs2);
            
            Object result;
            
            for (int i = 1; i <= 3; i++) {
				// Change the query text for each iteration.
                kwargs2.put("query_text", "A sentence " + i);
                
				// Load table
                table_obj = interpreter.invoke("worker", params1);
                
                kwargs2.put("table", table_obj);
                
				// Query the result
                result = interpreter.invoke("worker", params2);
                System.out.println(result);                
            }

        } catch (Exception e) {
            e.printStackTrace(); // Print any exceptions that occur
        }
    }
}
```

By default, the query function will return json records. We can get the returned result saved in arrow file as follows:


```java
public class QueryTable {

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
            
            // Step 2: query the table
            // Create the main HashMap for params2
            HashMap<String, Object> params2 = new HashMap<>();
            params2.put("task", "querying");
            params2.put("action", "query_from_table");
            
            
            // Create the kwargs2 HashMap
            HashMap<String, Object> kwargs2 = new HashMap<>();
            kwargs2.put("limit", 3);

            // filter the query output
			kwargs2.put("where", "(info = 'IBM')");
            
			// need to return arrow format
            HashMap<String, Object> return_format = new HashMap<>();
            return_format.put("format", "arrow");
            return_format.put("output", "path/to/output.arrow");
            return_format.put("batch_size", 1000);
            kwargs2.put("return_format", return_format); // arrow format
            
            params2.put("kwargs", kwargs2);
            
            Object result;
            
            for (int i = 1; i <= 3; i++) {
				// Change the query text for each iteration.
                kwargs2.put("query_text", "A sentence " + i);
                
				// Load table
                table_obj = interpreter.invoke("worker", params1);
                
                kwargs2.put("table", table_obj);
                
				// Query the result
                result = interpreter.invoke("worker", params2);
                System.out.println(result);                
            }

        } catch (Exception e) {
            e.printStackTrace(); // Print any exceptions that occur
        }
    }
}
```
