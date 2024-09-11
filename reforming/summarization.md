# Service Name Entity

1) Prepare the params files

```summarization.txt
{
  "task": "reforming",
  "action": "summarize_text",
  "kwargs": {
    "input_arrow": "summarize_text.arrow",
    "output_arrow": "/tmp/tmpm8jlvkqc",
    "model_name": "Falconsai/medical_summarization",
    "source_column": "text",
    "max_text_length": 200,
    "min_text_length": 50,
    "do_sample": false
  }
}
```

2) Now create a Java class to test it:

Notice: You can create a HashMap directly from Java, and you do not need to read the text file to create a map.

```JAVA
import pemja.core.PythonInterpreter;
import pemja.core.PythonException;
import pemja.core.PythonInterpreterConfig;
import pemja.core.object.PyObject;

import java.lang.reflect.Field;
import java.text.SimpleDateFormat;
import java.util.ArrayList;
import java.util.Date;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.Iterator;
import java.util.TimeZone;
import java.util.GregorianCalendar;
import java.util.Calendar;
import java.util.Date;
//import java.sql.Date;
import java.sql.Time;
import java.sql.Timestamp;
import java.time.LocalDate;
import java.time.LocalTime;
import java.time.LocalDateTime;

import org.json.simple.JSONObject;
import org.json.simple.parser.JSONParser;

import java.io.FileReader;

public class Summarization {
    public static void main(String[] args) {
        // Create and configure the Python interpreter
        PythonInterpreterConfig config = PythonInterpreterConfig
            .newBuilder()
            .setPythonExec("python") // Specify the Python executable
            //.addPythonPaths(path) // Uncomment and specify if you need to add custom paths
            .build();

        // Use try-with-resources to ensure proper resource management
        try (PythonInterpreter interpreter = new PythonInterpreter(config)) {
            // Execute Python code
            interpreter.exec("from titan.tbrane.services import worker");

            JSONParser parser = new JSONParser();

            try (FileReader reader = new FileReader("summarization.txt")) {
                // Parse the JSON file into JSONObject
                JSONObject jsonObject = (JSONObject) parser.parse(reader);

                // Convert JSONObject to a HashMap
                HashMap<String, Object> params = toMap(jsonObject);

                long startTime = System.currentTimeMillis();

                // Convert the HashMap to a Python dictionary (if needed by your Python API)
                Object result = interpreter.invoke("worker", params);

                long endTime = System.currentTimeMillis();
                
                // Calculate the elapsed time
                long elapsedTime = endTime - startTime;

                System.out.println("Execution time: " + elapsedTime + " milliseconds");
                System.out.println(result);

            } catch (Exception e) {
                e.printStackTrace(); // Exception for reader
            }

        } catch (Exception e) {
            e.printStackTrace(); // Exception for interpreter
        }
    }

    // Convert JSONObject to HashMap<String, Object>
    public static HashMap<String, Object> toMap(JSONObject jsonObject) {
        HashMap<String, Object> map = new HashMap<>();
        Iterator<?> keys = jsonObject.keySet().iterator();

        while (keys.hasNext()) {
            String key = (String) keys.next();
            Object value = jsonObject.get(key);
            map.put(key, value);
        }
        return map;
    }
}
```
