# Download Huggingface Models


The main source to download huggingface models


```JAVA
public class DownloadModel {
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
            kwargs.put("repo_id", "minishlab/M2V_base_output");
            kwargs.put("local_dir", "M2V_base_output");
            kwargs.put("force_download", false);
            kwargs.put("token", null);

            // Create the main HashMap for params
            HashMap<String, Object> params = new HashMap<>();
            params.put("task", "vending");
            params.put("action", "download_huggingface_model");
            params.put("kwargs", kwargs);
            interpreter.invoke("worker", params);
            
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
  "action": "download_huggingface_model",
  "kwargs": {
    "repo_id": String (id of model repo),
	"local_dir": String (path to saved folder),
	"force_download": Boolean (false),
	"token": String or Null (null),
    }
  }
}
```

Once we invoked the worker, the download process will start and save the model.

