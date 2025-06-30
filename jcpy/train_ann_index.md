# Train semantic search index


Java source sample:

```java
public class AnnIndexTrainTest extends BaseTest {

    private static final int VECTOR_SIZE = 768;
    private static final int NUM_VECTORS = 5;

    public static List<float[]> generateRandomVectors() {
        List<float[]> vectors = new ArrayList<>();
        Random random = new Random();

        for (int i = 0; i < NUM_VECTORS; i++) {
            float[] vector = new float[VECTOR_SIZE];
            for (int j = 0; j < VECTOR_SIZE; j++) {
                vector[j] = -random.nextFloat();  // generates value between -1.0 and 0.0
            }
            vectors.add(vector);
        }

        return vectors;
    }

    @Test
    public void testAnnTrain() {

        try (PythonInterpreter interpreter = new PythonInterpreter(config)) {
            interpreter.exec("from titan.tbrane.services import tbrane_multi_tasking");

            String inputArrow = projectDir + "/src/tests/resources/biorxiv.arrow";
            String outputArrow = projectDir + "/src/tests/resources/output.arrow";
            String indexFolder = projectDir + "/src/tests/resources/biorxiv_index";


            ArrayList<HashMap<String, Object>> jobs = new ArrayList<>();

            // Job 1: Deduplicate vectors
            jobs.add(
                JobBuilder.job()
                    .task("training")
                    .action("train_ann_index")
                    .kwargs(k -> {
                        k.put("input_arrow", inputArrow);
                        k.put("index_folder", indexFolder);
                        k.put("vector_name", "vector");
                        k.put("key_name", "paper_id");
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

- *input_arrow*: String (path to input arrow file)
- *index_folder*: String (Folder to save the index.bin and config file)
- *vector_name*: String (name of vector column)
- *key_name*: String (name of the key column)

