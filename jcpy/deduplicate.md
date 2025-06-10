# Deduplicate vectors

This is a new interface using JCPY.

Here is the Base class to config the main interpreter:

```java
package com.example;

import jcpy.engine.PythonInterpreterConfig;
import jcpy.engine.LogUtils;

import jcpy.engine.MainInterpreter;

import org.junit.jupiter.api.BeforeAll;
import org.junit.jupiter.api.AfterAll;
import java.util.logging.Logger;

public abstract class BaseTest {

    protected static final Logger logger = Logger.getLogger(BaseTest.class.getName());
    protected static PythonInterpreterConfig config;
    protected static String projectDir;
    protected static MainInterpreter mainInterpreter = MainInterpreter.instance;

    static {
        LogUtils.setupLogger();
    }

    @BeforeAll
    static void setUp() {
        String libLoaderPath = "/data/TitanProjects/jcpy/src/lib/jcpy_loader.so";
        String pythonLibPath = "/usr/lib/x86_64-linux-gnu/libpython3.10.so.1.0";
        String libEnginePath = "/data/TitanProjects/jcpy/src/lib/jcpy_engine.so";
        projectDir = System.getProperty("user.dir");
        config = PythonInterpreterConfig
                .newBuilder()
                .setPythonLibPath(pythonLibPath)
                .setLoaderLibPath(libLoaderPath)
                .setEngineLibPath(libEnginePath)
                .build();

        mainInterpreter.initialize(config);
    }
    
    @BeforeAll
    static void cleanUp() {
        mainInterpreter.close();
    }
}
```

Now you can use the Base class to run any other classes. Below is an example to run deduplicate vectors using the Base class above.

```java
package com.example;

import jcpy.engine.PythonInterpreter;
import jcpy.engine.PythonException;
import jcpy.engine.PythonInterpreterConfig;
import jcpy.engine.object.PyObject;

import jcpy.engine.PyUtils;
import jcpy.engine.JobBuilder;

import org.junit.jupiter.api.*;
import static org.junit.jupiter.api.Assertions.*;

import java.util.ArrayList;
import java.util.HashMap;



public class DedupVectorsTest extends BaseTest {

    @Test
    public void testDedupVectors() {

        try (PythonInterpreter interpreter = new PythonInterpreter(config)) {
            interpreter.exec("from titan.tbrane.services import tbrane_multi_tasking");

            String inputArrow = projectDir + "/src/tests/resources/duplicated_vectors.arrow";
            String outputArrow = projectDir + "/src/tests/resources/output.arrow";

            ArrayList<HashMap<String, Object>> jobs = new ArrayList<>();

            // Job 1: Deduplicate vectors
            jobs.add(
                JobBuilder.job()
                    .task("processing")
                    .action("deduplicate_vectors")
                    .kwargs(k -> {
                        k.put("input_arrow", inputArrow);
                        k.put("vector_name", "vector");
                        k.put("output_arrow", outputArrow);
                        k.put("vector_dims", 256);
                        k.put("max_k", 100);
                        k.put("threshold", 0.99);
						k.put("keep_vectors", true); // or false;
                    })
                    .build()
            );
            
            long startTime = System.currentTimeMillis();
            Object result = interpreter.invoke("tbrane_multi_tasking", jobs);
            long endTime = System.currentTimeMillis();

            logger.info("Testing time: " + (endTime - startTime) + " ms");
            logger.info("Returned object: " + String.valueOf(result));

        } catch (Exception e) {
            e.printStackTrace();
            Assertions.fail("Python interpreter failed: " + e.getMessage());
        }
    }

}
```

Một vài chú ý: `JobBuilder` và `PyUtils` là hai công cụ được hỗ trợ để hoạt động với các gói của iTitan. Thông qua nó, việc xây dựng các tham số rất đơn giản.


Giải thích tham số:

- *input_arrow*: Input arrow file
- *vector_name*: Name of the vector column
- *output_arrow*: Output arrow file
- *vector_dims*: Vector dimensions (INT)
- *max_k*: Max number of neighborhoods to search for similarity. 100 is good enough.
- *threshold*: The level of similarity to be removed: Loại các vector tương đồng 99% trở lên.
- 


