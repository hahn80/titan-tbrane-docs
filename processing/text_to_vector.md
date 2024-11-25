# Convert Text to Vector


To convert text to vector we need (a) load the encoder model and then (b) convert text to vector. If the distill model is used, please set is_distill=true!

```JAVA
public class Text2Vector {

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
            
            // Step 1: load encoder and table
            // params for loading encoder
            HashMap<String, Object> params = new HashMap<>();
            params.put("task", "loading");
            params.put("action", "load_sentence_encoder");
            
            
            // Create the nested HashMap for kwargs
            HashMap<String, Object> kwargs = new HashMap<>();
            kwargs.put("device", "cuda");
            
            // Load normal sentence transformer
            //kwargs.put("model_name_or_path", "/data/TitanProjects/dev/tbrane/bge-base-en-v1.5");
            //kwargs.put("is_distill", false);
            
            // Load normal distill model with is_distill = true
            kwargs.put("model_name_or_path", "/data/TitanProjects/dev/tbrane/M2V_base_output");
            kwargs.put("is_distill", true);
            
            params.put("kwargs", kwargs);
            
            long startTime = System.currentTimeMillis();
            
            Object encoder_obj = interpreter.invoke("worker", params);
            
            long endTime = System.currentTimeMillis();
            
            // Calculate the elapsed time
            long elapsedTime = endTime - startTime;
            
            System.out.println("Loading encoder: " + elapsedTime + " milliseconds");

            // params for loading table
            HashMap<String, Object> params1 = new HashMap<>();
            params1.put("task", "processing");
            params1.put("action", "convert_text_to_vector");
            
            HashMap<String, Object> kwargs1 = new HashMap<>();
            kwargs1.put("model_obj", encoder_obj);
            
            // Create a list of sentences
            List<String> sentences = new ArrayList<>();

            // Add two sentences to the list
            sentences.add("This is the first sentence.");
            sentences.add("Here is the second sentence.");

            kwargs1.put("sentences", sentences);
            
            params1.put("kwargs", kwargs1);
            
            Object result = interpreter.invoke("worker", params1);
            System.out.println(result);


        } catch (Exception e) {
            e.printStackTrace(); // Print any exceptions that occur
        }
    }
}
```

The structure of the *params* HashMap:

```JSON
{
  "task": "loading",
  "action": "load_sentence_encoder",
  "kwargs": {
    "model_name_or_path": String (path to the model),
    "device": String (cpu or cuda),
	"is_distill": Bool (true if distill model else false),
    }
  }
}
```
Notice: load the encoder_obj each time you want to vectorize sentences.

