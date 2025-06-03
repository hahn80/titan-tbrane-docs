# Chonkie Texts with Py4J

1. Setup Python bridge: `jp_bridge.py`

```python
import gc
import sys
import signal
import array
import hashlib
import numpy as np
from typing import Union, Literal

from py4j.clientserver import ClientServer, JavaParameters, PythonParameters

from sentence_transformers import SentenceTransformer
from titan.tbrane.embeddings.models import Model2VecEmbeddings
from titan.tbrane.vender.chonkie.embeddings import SentenceTransformerEmbeddings
from titan.tbrane.vender.model2vec import StaticModel
from titan.tbrane.vender.chonkie import (
    SemanticChunker,
    SDPMChunker,
    LateChunker,
)
from titan.tbrane.tasks.processing import deduplicate_texts


class VectorModel:
    def __init__(self, model):
        self.model = model

    def to_vector(self, text: str) -> bytearray:
        embeddings = self.model.encode(text)
        numpy_matrix = embeddings.astype(np.float32)
        return self.to_buffer(numpy_matrix)

    def to_chunk(
        self,
        text,
        threshold: Union[str, float, int] = "auto",
        chunk_size: int = 512,
        strategy: Literal["semantic", "sdpm", "late"] = "semantic",
        similarity_window: int = 1,
        min_sentences: int = 1,
        min_chunk_size: int = 2,
        min_characters_per_sentence: int = 12,
        threshold_step: float = 0.01,
    ) -> str:
        if strategy == "late":
            model = SentenceTransformerEmbeddings(self.model)
        else:
            model = Model2VecEmbeddings(self.model)
        chunker_supported_models = {
            "semantic": SemanticChunker,
            "sdpm": SDPMChunker,
            "late": LateChunker,
        }
        if strategy not in chunker_supported_models:
            raise ValueError(f"Unsupported chunker strategy: {strategy}")
        chunker_class = chunker_supported_models[strategy]
        chunker = chunker_class(
            embedding_model=model,
            chunk_size=chunk_size,
            threshold=threshold,
            similarity_window=similarity_window,
            min_sentences=min_sentences,
            min_chunk_size=min_chunk_size,
            min_characters_per_sentence=min_characters_per_sentence,
            threshold_step=threshold_step,
            strategy=strategy,
        )
        chunks = chunker.chunk(text)
        py_list = [chunk.text for chunk in chunks if chunk.text is not None]
        return "__sp@ce__".join(py_list)

    def remove_duplicates(self, texts, threshold) -> str:
        py_list = deduplicate_texts(self.model, texts, threshold=threshold)
        return "__sp@ce__".join(py_list)

    @staticmethod
    def to_buffer(numpy_matrix) -> bytearray:
        header = array.array("i", list(numpy_matrix.shape))
        body = array.array("f", numpy_matrix.flatten().tolist())

        if sys.byteorder != "big":
            header.byteswap()
            body.byteswap()

        buf = bytearray(header.tobytes() + body.tobytes())
        return buf

    class Java:
        implements = ["com.example.VectorModel"]


class SentenceEmbedding:
    embedding_models = {}

    def _hashing(self, args) -> str:
        text = "_".join(args)
        return hashlib.md5(text.encode("utf-8")).hexdigest()

    def static_model(self, model_name_or_path: str) -> VectorModel:
        # print(f"Starting model with name/path: {model_name_or_path}")
        model_hashed = self._hashing(("static", model_name_or_path))
        if model_hashed not in self.embedding_models:
            model = StaticModel.from_pretrained(path=model_name_or_path)
            self.embedding_models[model_hashed] = VectorModel(model)

        return self.embedding_models[model_hashed]

    def transformer_model(self, model_name_or_path: str, backend) -> VectorModel:
        # print(f"Starting model with name/path: {model_name_or_path}")
        model_hashed = self._hashing(("transformer", model_name_or_path, backend))
        if model_hashed not in self.embedding_models:
            model = SentenceTransformer(
                model_name_or_path=model_name_or_path, backend=backend
            )
            self.embedding_models[model_hashed] = VectorModel(model)

        return self.embedding_models[model_hashed]

    def destroy_embedding_models(self):
        for _, model in self.embedding_models.items():
            del model
        self.embedding_models.clear()
        gc.collect()

    class Java:
        implements = ["com.example.SentenceEmbedding"]


def shutdown_handler(signum, frame):
    print("\nShutting down Py4J server gracefully...")
    server.shutdown()


if __name__ == "__main__":
    print("Python ClientServer is running. Press Ctrl+C to exit.")
    signal.signal(signal.SIGINT, shutdown_handler)

    server = ClientServer(
        java_parameters=JavaParameters(auto_convert=True),
        python_parameters=PythonParameters(),
        python_server_entry_point=SentenceEmbedding(),
    )
```


2. Java main classes:

```java
package com.example;

public interface SentenceEmbedding {
    VectorModel static_model(String model_name_or_path);
    VectorModel transformer_model(String model_name_or_path, String backend);
    void destroy_embedding_models();
}
```

```java
package com.example;

import java.util.ArrayList;


public interface VectorModel {
    byte[] to_vector(String text);

    String to_chunk(String text, String threshold, int chunk_size, String strategy);
    String to_chunk(String text, float threshold, int chunk_size, String strategy);
    String to_chunk(String text, double threshold, int chunk_size, String strategy);
}
```



3. Java test function:

```java
package com.example;

import java.util.logging.Logger;
import jcpy.engine.LogUtils;

import py4j.ClientServer;
import py4j.Py4JNetworkException;

import java.nio.ByteBuffer;
import java.nio.ByteOrder;
import java.util.Arrays;
import java.util.ArrayList;


import org.junit.jupiter.api.*;
import static org.junit.jupiter.api.Assertions.assertTrue;
import static org.junit.jupiter.api.Assumptions.assumeTrue;
import org.junit.BeforeClass;
import java.io.*;


public class ChonkieTest {

    private static final Logger logger = Logger.getLogger(ChonkieTest.class.getName());
    
    private static boolean setupSuccessful = false; // Flag to track setup status
    
    private static Process pythonProcess;
    
    protected static String projectDir = System.getProperty("user.dir");
    
    static {
        LogUtils.setupLogger();
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
    
    private static ClientServer clientServer;
    private static SentenceEmbedding embed;
    
    @BeforeAll
    public static void setup() throws Exception {
    
        String bridgePath = projectDir + "/src/tests/python/jp_bridge.py";
    
        ProcessBuilder pb = new ProcessBuilder("python", bridgePath);
        pb.redirectErrorStream(true);
        pythonProcess = pb.start();
        
        logger.info("Start Python server ...");

        // Optional: wait for the script to be ready
        try { Thread.sleep(5000); } catch (InterruptedException e) { e.printStackTrace(); }
        

        // Setup ClientServer and SentenceEmbedding
        try {
            clientServer = new ClientServer(null);
            setupSuccessful = true; // Set flag to true only if setup succeeds
        } catch (Py4JNetworkException e) {
            logger.severe("Error setting up Py4J ClientServer: " + e.getMessage());
            setupSuccessful = false; // Set flag to false on failure
            //throw new Exception("Error setting up Py4J ClientServer.", e);
        }
    }
    
    @AfterAll
    public static void cleanup() {
        if (clientServer != null) {
            clientServer.shutdown();
            logger.info("ClientServer shutdown complete.");
        }
        
        if (pythonProcess != null && pythonProcess.isAlive()) {
            pythonProcess.destroy(); // sends SIGTERM
            try {
                pythonProcess.waitFor(); // wait for it to exit
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }


    @Test
    public void testChunk() {
    
        assumeTrue(setupSuccessful, "Skipping test due to setup failure.");
        
        long startTime = System.currentTimeMillis();
        
        embed = (SentenceEmbedding) clientServer.getPythonServerEntryPoint(new Class[]{SentenceEmbedding.class});

        // Load static model for testing performance
        //VectorModel model = embed.static_model("/data/TitanProjects/dev/tbrane/potion-base-8M");
		//String strategy = "semantic";
		//String strategy = "sdpm";
		//String threshold = "auto";
		//Int chunk_size = 256;

        // Load sentence transformer model for LATE:
        VectorModel model = embed.transformer_model("all-MiniLM-L6-v2", "onnx");
		String strategy = "late";

        String resulstStrings = model.to_chunk(
                "Ủy viên Bộ Chính trị, Thủ tướng Chính phủ Phạm Minh Chính vừa ký Chỉ thị số 07/CT-TTg đẩy mạnh triển khai Đề án phát triển ứng dụng dữ liệu về dân cư, định danh và xác thực điện tử phục vụ chuyển đổi số quốc gia. Thủ tướng Chính phủ giao Bộ Tài chính xây dựng Kế hoạch đẩy mạnh triển khai Chỉ thị 18/CT-TTg ngày 30.5.2023 của Thủ tướng Chính phủ phục vụ phát triển thương mại điện tử, chống thất thu thuế, đảm bảo an ninh tiền tệ, gắn trách nhiệm cụ thể từng bộ ngành để tổ chức triển khai thực hiện, hoàn thành trong tháng 3.2025. Xây dựng, tham mưu Chính phủ trình Quốc hội ban hành Nghị quyết quy định về cơ chế đặc thù đầu tư, đầu tư công, mua sắm công các sản phẩm, dịch vụ số để thúc đẩy mạnh mẽ chuyển đổi số quốc gia giai đoạn 2025-2030, báo cáo Chính phủ trong tháng 5.2025.",
                threshold, chunk_size, strategy);
        String[] parts = resulstStrings.split("__sp@ce__");
        ArrayList<String> results = new ArrayList<>(Arrays.asList(parts));
        System.out.println(results);

        long endTime = System.currentTimeMillis();
        double elapsedTimeMs = (endTime - startTime);

        logger.info(String.format("Chunking Elapsed time: %.2f ms", elapsedTimeMs));

    }

}
```



