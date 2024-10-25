# JCPY General Guidelines


Jcpy is a multipurpose function calling to get access to CPython.
Here is the first examples from Java side:

Import the following packages:
```java
import jcpy.engine.PythonInterpreter;
import jcpy.engine.PythonException;
import jcpy.engine.PythonInterpreterConfig;
import jcpy.engine.object.PyObject;

import java.lang.reflect.Field;
import java.text.SimpleDateFormat;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.TimeZone;
import java.util.GregorianCalendar;
import java.util.Calendar;
//import java.util.Date;
import java.sql.Date;
import java.sql.Time;
import java.sql.Timestamp;
import java.time.LocalDate;
import java.time.LocalTime;
import java.time.LocalDateTime;
```


The main content of the Java source:

```java
public class FirstRun {
    public static void main(String[] args) {
        // Configure the Python interpreter
        PythonInterpreterConfig config = PythonInterpreterConfig
            .newBuilder()
            .setPythonExec("python") // Specify the Python executable
            //.addPythonPaths(path) // Specify if you need to add custom paths
            .build();

        // Use try-with-resources to ensure proper resource management
        try (PythonInterpreter interpreter = new PythonInterpreter(config)) {
            // Execute Python code
            interpreter.exec("from titan.tbrane import get_jcpy_path");
            
            Object path = interpreter.invoke("get_jcpy_path");
            
            // Now you can print out the object
            System.out.println(path);
        } catch (Exception e) {
            e.printStackTrace(); // Print any exceptions
        }
    }
}
```
