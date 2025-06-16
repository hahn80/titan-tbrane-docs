# Search vector directly from arrow


Java source sample:

```java
public class VectorSearchTest extends BaseTest {

    public static float[] generateRandomVector() {
        Random random = new Random();
        float[] vector = new float[128];
        for (int i = 0; i < vector.length; i++) {
            vector[i] = random.nextFloat(); // Generates float between 0.0 (inclusive) and 1.0 (exclusive)
        }
        return vector;
    }

    @Test
    public void testVectorSearch() {

        try (PythonInterpreter interpreter = new PythonInterpreter(config)) {
            interpreter.exec("from titan.tbrane.services import tbrane_multi_tasking");

            String inputArrow = projectDir + "/src/tests/resources/arrow_search.arrow";
            String outputArrow = projectDir + "/src/tests/resources/output.arrow";
            float[] queryPoint = generateRandomVector();

            ArrayList<HashMap<String, Object>> jobs = new ArrayList<>();

            // Job 1: Deduplicate vectors
            jobs.add(
                JobBuilder.job()
                    .task("querying")
                    .action("arrow_search")
                    .kwargs(k -> {
                        k.put("input_arrow", inputArrow);
                        k.put("vector_name", "vectors");
                        k.put("query_point", queryPoint);
                        k.put("k", 10);
                        k.put("metric", "euclidean");
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

Giải thích tham số:

- *input_arrow*: String (path to arrow file)
- *vector_name*: String (name of vector column)
- *query_point*: Float [], the array of query vector (float 32)
- *k*: the number of vectors to collect from the search. If k=10, it means that we look for 10 nearest vector to *query_point*.
- *metric*: type of distance to use for searching: There are 4 types: "euclidean", "manhattan", "cosine_similarity", "inner_product"

