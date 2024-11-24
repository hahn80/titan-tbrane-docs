# JCPY Welcome Testing


Jcpy welcome function to invoke the worker service:

```java
import jcpy.engine.PythonInterpreter;
import jcpy.engine.PythonException;
import jcpy.engine.PythonInterpreterConfig;
import jcpy.engine.object.PyObject;
```


The main content of the Java source:

```java
public class Welcome {
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
            kwargs.put("name", "Titan");

            // Create the main HashMap for params
            HashMap<String, Object> params = new HashMap<>();
            params.put("task", "testing");
            params.put("action", "welcome");
            params.put("kwargs", kwargs);
            
            
            Object text_obj = interpreter.invoke("worker", params);
                
            // Now you can use the object
            System.out.println(text_obj);

            
        } catch (Exception e) {
            e.printStackTrace(); // Print any exceptions
        }
    }
}
```
