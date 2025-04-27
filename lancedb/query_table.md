# Query Vector DB

Once the vector database is created, we now can query the database to get the result for RAG model.

```JAVA
public class QueryVectorTable {

    private static final Random random = new Random();

    public static void main(String[] args) {
        // Create and configure the Python interpreter
        PythonInterpreterConfig config = PythonInterpreterConfig
            .newBuilder()
            .setPythonExec("python") // Specify the Python executable
            //.addPythonPaths(path) // Uncomment and specify if you need to add custom paths
            .build();

        // Use try-with-resources to ensure proper resource management
        try (PythonInterpreter interpreter = new PythonInterpreter(config)) {
            // Import Python tbrane_worker function
            interpreter.exec("from titan.tbrane.services import tbrane_worker");
            
            
            
            Map<String, Object> kwargs = new HashMap<>();
            
            kwargs.put("database_path", "test_db");
            kwargs.put("table_name", "test");
            kwargs.put("vector", randomVector(256));
            kwargs.put("select_columns", new ArrayList<>(List.of("key")));
            kwargs.put("limit", 3);
            //kwargs.put("where", "(info = 'IBM')");

            // Nested map for return_format
            Map<String, Object> returnFormat = new HashMap<>();
            returnFormat.put("format", "arrow");
            returnFormat.put("output", "output.arrow");
            returnFormat.put("batch_size", 10);

            // Disable returnFormat to get json return
			kwargs.put("return_format", returnFormat);
            
            // Create the main HashMap for params
            HashMap<String, Object> params = new HashMap<>();
            params.put("task", "querying");
            params.put("action", "query_vector_table");
            params.put("kwargs", kwargs);

            
            long startTime = System.currentTimeMillis();
            
            Object result = interpreter.invoke("tbrane_worker", params);
            
            long endTime = System.currentTimeMillis();
            
            // Calculate the elapsed time
            long elapsedTime = endTime - startTime;
            
            System.out.println("Loading table: " + elapsedTime + " milliseconds");
            
            System.out.println(result);


        } catch (Exception e) {
            e.printStackTrace(); // Print any exceptions that occur
        }
    }
    
    
    
    private static ArrayList<Float> randomVector(int size) {
        ArrayList<Float> vector = new ArrayList<>(size);
        for (int i = 0; i < size; i++) {
            float value = -1.0f + random.nextFloat() * 2.0f;
            vector.add(value);
        }
        return vector;
    }
}
```

Notice: You can get json return instead of arrow output if you disable kwargs.put("return_format", returnFormat); from the kkwargs map.
