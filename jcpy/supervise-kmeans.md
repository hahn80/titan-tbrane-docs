# Supervised Kmeans

1. Training the supervised kmeans model:

This java code runs with jpcy 1.0.2, but it can be adjusted to use with 1.0.0

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
import java.util.Date;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.Arrays;
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


public class KmeansTrainTest extends BaseTest {

    @Test
    public void testKmeansTrain() {

        try (PythonInterpreter interpreter = new PythonInterpreter(config)) {
            interpreter.exec("from titan.tbrane.services import tbrane_multi_tasking");

            String inputArrow = projectDir + "/src/tests/resources/vectors.arrow";
            String outputKmeans = projectDir + "/src/tests/resources/kmeans.bin";

            ArrayList<HashMap<String, Object>> jobs = new ArrayList<>();

            // Job 1: Deduplicate vectors
            jobs.add(
                JobBuilder.job()
                    .task("training")
                    .action("supervised_kmeans")
                    .kwargs(k -> {
                        k.put("arrow_file", inputArrow);
                        k.put("n_clusters", 20);
                        k.put("vector_column", "vector");
                        k.put("saved_model_path", outputKmeans);
                    }).build()
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

Giải thích tham số:

- `arrow_file`: File arrow mô tả thuộc tính của các nhóm. Tập hợp n vectors theo thứ tự nhất định.
- `n_clusters`: Số nhóm được mô tả để clustering. Chú ý số nhóm `n_clusters` phải bằng số vectors mô tả tronng file input arrow. Nếu số `n_clusters` không trùng khớp với số lượng vector, sẽ báo lỗi.


2. Predict the group label:

Sau khi train model xong, chúng ta có thể predict như model Kmean bình thường.

```java
public class KmeansPredictTest extends BaseTest {

    @Test
    public void testKmeansPredict() {

        try (PythonInterpreter interpreter = new PythonInterpreter(config)) {
            interpreter.exec("from titan.tbrane.services import tbrane_multi_tasking");

            String inputArrow = projectDir + "/src/tests/resources/vectors.arrow";
            String outputArrow = projectDir + "/src/tests/resources/output.arrow";
            String outputKmeans = projectDir + "/src/tests/resources/kmeans.bin";

            ArrayList<HashMap<String, Object>> jobs = new ArrayList<>();

            // Job 1: Deduplicate vectors
            jobs.add(
                JobBuilder.job()
                    .task("predicting")
                    .action("predict_cluster_group")
                    .kwargs(k -> {
                        k.put("arrow_file", inputArrow);
                        k.put("saved_model_path", outputKmeans);
                        k.put("output_file", outputArrow);
                        k.put("key_column", "key");
                        k.put("vector_column", "vector");
                        k.put("cluster_column", "cluster");
                    }).build()
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

