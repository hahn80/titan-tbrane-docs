# Semantic search index


Once we train the index, we can load it and search as many times we want.

```java
public class AnnIndexSearchTest extends BaseTest {

    private static final int VECTOR_SIZE = 768;
    private static final int NUM_VECTORS = 5;

    public static ArrayList<float[]> generateRandomVectors() {
        ArrayList<float[]> vectors = new ArrayList<>();
        Random random = new Random();

        for (int i = 0; i < NUM_VECTORS; i++) {
            float[] vector = new float[VECTOR_SIZE];
            for (int j = 0; j < VECTOR_SIZE; j++) {
                vector[j] = random.nextFloat();  // generates value between -1.0 and 0.0
            }
            vectors.add(vector);
        }

        return vectors;
    }


    @Test
    public void testAnnSearch() {

        try (PythonInterpreter interpreter = new PythonInterpreter(config)) {
            interpreter.exec("from titan.tbrane.services import tbrane_multi_tasking");

            String inputArrow = projectDir + "/src/tests/resources/biorxiv.arrow";
            String outputArrow = projectDir + "/src/tests/resources/output.arrow";
            String indexFolder = projectDir + "/src/tests/resources/biorxiv_index";
            ArrayList<float[]> queryVectors = generateRandomVectors();

            ArrayList<HashMap<String, Object>> jobs = new ArrayList<>();

            // Job 1: Deduplicate vectors
            jobs.add(
                JobBuilder.job()
                    .task("querying")
                    .action("search_ann_index")
                    .kwargs(k -> {
                        k.put("index_folder", indexFolder);
                        k.put("query_vectors", queryVectors);
                        k.put("limit", 3);
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

- *index_folder*: String (Folder to save the index.bin and config file)
- *query_vectors*: Array of float vector, the length of each vector must be the same as the original arrow file that is used for training.
- *limit*: Int (the number of matched result to search for)

