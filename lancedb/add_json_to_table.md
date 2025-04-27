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
            interpreter.exec("from titan.tbrane.services import tbrane_worker");
            
            // Create the HashMap for kwargs
            HashMap<String, Object> kwargs = new HashMap<>();
            kwargs.put("database_path", "test_db");
            kwargs.put("table_name", "test");
            
            List<Map<String, Object>> records = generateRecords(10);
            
            kwargs.put("records", records);
            
            kwargs.put("mode", "append"); //or 

            // Create the main HashMap for params
            HashMap<String, Object> params = new HashMap<>();
            params.put("task", "creating");
            params.put("action", "add_json_to_table");
            params.put("kwargs", kwargs);
            
            
            Object table_obj = interpreter.invoke("tbrane_worker", params);
            System.out.println(table_obj);

        } catch (Exception e) {
            e.printStackTrace(); // Print any exceptions
        }
    }
    
    private static final Random random = new Random();
    private static final String LETTERS = "abcdefghijklmnopqrstuvwxyz";
    
    private static String randomLetter() {
        int index = random.nextInt(LETTERS.length());
        return String.valueOf(LETTERS.charAt(index));
    }

    private static String randomString(int length) {
        StringBuilder sb = new StringBuilder(length);
        for (int i = 0; i < length; i++) {
            sb.append(LETTERS.charAt(random.nextInt(LETTERS.length())));
        }
        return sb.toString();
    }

    private static ArrayList<Float> randomVector(int size) {
        ArrayList<Float> vector = new ArrayList<>(size);
        for (int i = 0; i < size; i++) {
            float value = -1.0f + random.nextFloat() * 2.0f;
            vector.add(value);
        }
        return vector;
    }
    
    public static List<Map<String, Object>> generateRecords(int count) {
        List<Map<String, Object>> records = new ArrayList<>();

        for (int i = 0; i < count; i++) {
            Map<String, Object> record = new HashMap<>();

            record.put("key", random.nextInt(1_000_000));
            record.put("text", randomLetter());
            record.put("info", randomString(10));
            record.put("vector", randomVector(256));

            records.add(record);
        }

        return records;
    }
}
```

The structure of the *params* HashMap:

```JSON
{
  "task": "creating",
  "action": "add_json_to_table",
  "kwargs": {
    "database_path": database_path,
	"table_name": table_name,
	"mode": "append",
    "records": [
      {
        "key": 10,
		  "text": "Some text",
        "vector": [....],
        "info": "IBM"
      },
      {
        "key": 11,
		  "text": "Some text",
        "vector": [....],
        "info": "MSFT"
      }
    ]
  }
}

```

Notice: we can remove the "text" column, it will be ok.

"task": (String) must be an existing task: `creating`.

"action": (String) must be supported actions. Supported: `add_json_to_table`;

"kwargs": another HashMap to pass the arguments to action function: `add_json_to_table`.

Set json data records
kwargs.put("records", list of HashMaps);

Once we invoked the worker, we now can use *table_obj* for the next computation.

