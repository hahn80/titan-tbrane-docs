# Text to Vector


In this document, we will use Py4J to load VectorModel and then convert any text to vector

Here is the sample code

```java
public class PythonJava {
    // Expose Python entry point to VectorModel
    public interface VectorModel {
        byte[] to_vector(String text);
    }

    // Expose Python entry point to VectorEmbedding
    public interface SentenceEmbedding {
        VectorModel static_model(String model_name_or_path);
        VectorModel transformer_model(String model_name_or_path, String backend);
        void destroy_embedding_models();
    }


    public static ArrayList<Float> parseVector(byte[] buf) {
        ByteBuffer byteBuffer = ByteBuffer.wrap(buf);
        byteBuffer.order(ByteOrder.BIG_ENDIAN);

        int dim = byteBuffer.getInt();
        ArrayList<Float> vector = new ArrayList<>(dim);
        for (int i = 0; i < dim; i++) {
            float value = byteBuffer.getFloat();
            vector.add(value);      
        }

        return vector;
    }


    public static void main(String[] args) {
        try {


            ClientServer clientServer = new ClientServer(null);

            SentenceEmbedding embed = (SentenceEmbedding) clientServer.getPythonServerEntryPoint(new Class[]{SentenceEmbedding.class});
            long startTime = System.nanoTime();
            
            // Load static model from model2vec
            VectorModel model = embed.static_model("/data/TitanProjects/dev/tbrane/M2V_base_output");
            
            // Load sentence transformer model
            //VectorModel model = embed.transformer_model("all-MiniLM-L6-v2", "torch");
            //VectorModel model = embed.transformer_model("all-MiniLM-L6-v2", "onnx");
            //VectorModel model = embed.transformer_model("all-MiniLM-L6-v2", "openvino");
            //VectorModel model = embed.transformer_model("/data/TitanProjects/dev/tbrane/bge-base-en-v1.5", "torch");
            
            long endTime = System.nanoTime();
            double elapsedTimeMs = (endTime - startTime) / 1_000_000.0;
            
            System.out.printf("Loading model: %.2f ms%n", elapsedTimeMs);
            
            int NUM_THREADS = 100;
            
            startTime = System.nanoTime();
                
            for (int i = 1; i <= NUM_THREADS; i++) {
                
                byte[] resultBytes = model.to_vector("Sample sentence here" + i);
                ArrayList<Float> vector = parseVector(resultBytes);
                //System.out.println("Vector length: " + vector.length);
            }
            
            endTime = System.nanoTime();
            elapsedTimeMs = (endTime - startTime) / 1_000_000.0;

            System.out.printf("Total Elapsed time: %.2f ms%n", elapsedTimeMs);
            
            // Many models loaded into memory, we can clear them up.
            //embed.destroy_embedding_models();

            clientServer.shutdown();

        } catch (Py4JNetworkException e) {
            e.printStackTrace();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

Notice: When you load many models on the memory, it keeps the model inside the memory so you can load it faster. If you want to free memory, you have to call
embed.destroy_embedding_models();
