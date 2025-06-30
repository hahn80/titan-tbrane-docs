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


Returned result: We can query multiple vector at once, each vector will give a search result as a Hashmap of keys and scores.

```json
[{scores=[0.15423892438411713, 0.15275096893310547, 0.1520572155714035], keys=[38c35aed007b69cf403a097e93b18ad261072568, b026bbd2c2afbbf5f36cfdae973657aaca4153a7, 13bf9fe0df944c0ee0debb0d445527ad4418cc19]}, {scores=[0.17349645495414734, 0.16804325580596924, 0.16546273231506348], keys=[c8559b6ddd6213bc91524397933bce33aea7d50f, 46c5f4a01496ab4fe89b1dfba7b7f94445766640, a8573e6963125b8842707d449bf5232185042208]}, {scores=[0.1558837890625, 0.15479260683059692, 0.15279778838157654], keys=[13bf9fe0df944c0ee0debb0d445527ad4418cc19, f6cadf1f17cc7bd098174c1428a459aff01aa4a5, 9ca81dfa68fcbff13166d810aca929027bb3763b]}, {scores=[0.16315773129463196, 0.15751829743385315, 0.15564846992492676], keys=[b026bbd2c2afbbf5f36cfdae973657aaca4153a7, e367637e063ec728b13f3685e8aee5f775c553cd, 622580ea34c3acc011faa7573bc549e34078edde]}, {scores=[0.14979572594165802, 0.14649449288845062, 0.1454213261604309], keys=[20154fa12ca525582cd1b42019927e8f7816fb51, 30457ee3ce0001f33938fbc246b4ce4eacd74f5d, e7ff9455c483c237c3ee08b9d230b7be65c064bf]}]
```
