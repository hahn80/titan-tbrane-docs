# Semantic search vector directly from arrow


Java source sample:

```java
public class VectorSemanticSearchTest extends BaseTest {

    public static float[] generateRandomVector() {
        Random random = new Random();
        float[] vector = new float[128];
        for (int i = 0; i < vector.length; i++) {
            vector[i] = random.nextFloat(); // Generates float between 0.0 (inclusive) and 1.0 (exclusive)
        }
        return vector;
    }

    @Test
    public void testVectorSemanticSearch() {

        try (PythonInterpreter interpreter = new PythonInterpreter(config)) {
            interpreter.exec("from titan.tbrane.services import tbrane_multi_tasking");

            String inputArrow = projectDir + "/src/tests/resources/biorxiv.arrow";
            String outputArrow = projectDir + "/src/tests/resources/output.arrow";
            float[] queryPoint = generateRandomVector();

            ArrayList<HashMap<String, Object>> jobs = new ArrayList<>();

            // Job 1: Deduplicate vectors
            jobs.add(
                JobBuilder.job()
                    .task("querying")
                    .action("arrow_semantic_search")
                    .kwargs(k -> {
                        k.put("input_arrow", inputArrow);
                        k.put("output_arrow", outputArrow);
                        k.put("vector_name", "vectors");
                        k.put("vector_length", 768);
                        k.put("query_vector", queryPoint);
                        k.put("limit", 10);
                        k.put("keep_vectors", true);
                        k.put("min_score", 0.7);
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
            // Assertions.fail("Python interpreter failed: " + e.getMessage());
        }
    }

}
```

Giải thích tham số:

- *input_arrow*: String (path to input arrow file)
- *output_arrow*: String (path to output arrow file)
- *vector_length*: String (len of each vector from embedding model)
- *vector_name*: String (name of vector column)
- *query_vector*: Float [], the array of query vector (float 32)
- *limit*: the number of vectors to collect from the search. If limit=10, it means that we look for 10 nearest vector to *query_vector*.
- *keep_vectors*: Bool: keep vector column or not.
- *min_score*: min_score to keep result.

