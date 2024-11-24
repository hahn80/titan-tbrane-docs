# Convert Markdown to Json


To use this function, make sure to install `pip install markdown_to_json`.

The main source to convert makrdown to json


```JAVA
public class MarkdownToJson {
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
            kwargs.put("data", "# Description \n\r \n\r This is an example file \n\r \n\r # Authors \n\r \n\r * Nate Vack\n\r * Vendor Packages\n\r * docopt\n\r * CommonMark-py \n\r \n\r # Versions \n\r \n\r ## Version 1 \n\r \n\r Here's something about Version 1; I said \"Hooray!\" \n\r \n\r ## Version 2 \n\r \n\r Here's something about Version 2");

            // Create the main HashMap for params
            HashMap<String, Object> params = new HashMap<>();
            params.put("task", "vending");
            params.put("action", "markdown_to_json");
            params.put("kwargs", kwargs);
            
            
            Map<String, Object> json_obj = (Map<String, Object>) interpreter.invoke("worker", params);

            // Now you can get access to the object
            System.out.println(json_obj);
        } catch (Exception e) {
            e.printStackTrace(); // Print any exceptions
        }
    }
}
```

The structure of the *params* HashMap:

```JSON
{
  "task": "vending",
  "action": "markdown_to_json",
  "kwargs": {
    "data": String: markdown json here
    }
  }
}
```

Once we invoked the worker, we now can get the json data of the input markdown.

