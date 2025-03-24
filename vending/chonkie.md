# Chonkie to chunk texts

Integrate 3 good chunking methods from chonkie: https://github.com/chonkie-ai/chonkie

1. SemanticChunker: Splits text into chunks based on semantic similarity.

```JAVA
public class ChunkText {

    public static void main(String[] args) {
        // Create and configure the Python interpreter
        PythonInterpreterConfig config = PythonInterpreterConfig
            .newBuilder()
            .setPythonExec("python") // Specify the Python executable
            //.addPythonPaths(path) // Uncomment and specify if you need to add custom paths
            .build();

        // Use try-with-resources to ensure proper resource management
        try (PythonInterpreter interpreter = new PythonInterpreter(config)) {
            // Import Python worker function
            interpreter.exec("from titan.tbrane.services import worker");
            
            // Step 1: load encoder and table
            // params for loading encoder
            HashMap<String, Object> params = new HashMap<>();
            params.put("task", "processing");
            params.put("action", "chonkie_chunk");
            
            
            // Create the nested HashMap for kwargs
            HashMap<String, Object> kwargs = new HashMap<>();
            
            kwargs.put("model_name_or_path", "/data/TitanProjects/dev/tbrane/M2V_base_output");
			kwargs.put("strategy", "semantic");
            kwargs.put("chunk_size", 128);
            kwargs.put("threshold", 0.25);
            kwargs.put("text", "The process of text chunking in RAG applications represents a delicate balance between competing requirements. On one side, we have the need for semantic coherence – ensuring that each chunk maintains meaningful context that can be understood and processed independently. On the other, we must optimize for information density, ensuring that each chunk carries sufficient signal without excessive noise that might impede retrieval accuracy. In this post, we explore the challenges of text chunking in RAG applications and propose a novel approach that leverages recent advances in transformer-based language models to achieve a more effective balance between these competing requirements.");
            
            params.put("kwargs", kwargs);
            
            long startTime = System.currentTimeMillis();
            
            Object result = interpreter.invoke("worker", params);
            
            long endTime = System.currentTimeMillis();
            
            // Calculate the elapsed time
            long elapsedTime = endTime - startTime;
            
            System.out.println("Loading encoder: " + elapsedTime + " milliseconds");

            
            System.out.println(result);


        } catch (Exception e) {
            e.printStackTrace(); // Print any exceptions that occur
        }
    }
}
```

The structure of the *params* HashMap:
`strategy`: String -> one of the 3 methods: "semantic", "sdpm", "late"
`chunk_size`: The size of chunked sentences (Integer)
`threshold`: The the threshold to ignore similarity (Float 0 < similarity_threshold < 1). It is better to set the threshold from 0.2 to 0.5).



2. SDPMChunker: Splits text using a Semantic Double-Pass Merge approach.

Similar to semantic, just change:
**kwargs.put("strategy", "semantic"); -> kwargs.put("strategy", "sdpm");**

3. LateChunker (experimental): Embeds text and then splits it to have better chunk embeddings.
Cannot use this function with DISTILLED MODELS, use original sentence transformer models, for example:

Convert the following params to Hashmap

```json
{
        "task": "vending",
        "action": "chonkie_chunk",
        "kwargs": {
            "text": text,
            "model_name_or_path": "all-MiniLM-L6-v2",
            "chunk_size": 256,
            "strategy": "late",
        },
    }
```



