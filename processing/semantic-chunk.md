# Semantic Chunk Text


To chunk sentences into groups of similar meanings.

```JAVA
public class SemanticChunk {

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
            params.put("action", "semantic_chunk");
            
            
            // Create the nested HashMap for kwargs
            HashMap<String, Object> kwargs = new HashMap<>();
            
            kwargs.put("model_name_or_path", "bge-m3");
            kwargs.put("host", "localhost:11434");
            kwargs.put("threshold_percentile", 90);
            kwargs.put("window_size", 2);
            kwargs.put("text", "Ủy viên Bộ Chính trị, Thủ tướng Chính phủ Phạm Minh Chính vừa ký Chỉ thị số 07/CT-TTg đẩy mạnh triển khai Đề án phát triển ứng dụng dữ liệu về dân cư, định danh và xác thực điện tử phục vụ chuyển đổi số quốc gia.    Thủ tướng Chính phủ giao Bộ Tài chính xây dựng Kế hoạch đẩy mạnh triển khai Chỉ thị 18/CT-TTg ngày 30.5.2023 của Thủ tướng Chính phủ phục vụ phát triển thương mại điện tử, chống thất thu thuế, đảm bảo an ninh tiền tệ, gắn trách nhiệm cụ thể từng bộ ngành để tổ chức triển khai thực hiện, hoàn thành trong tháng 3.2025.    Xây dựng, tham mưu Chính phủ trình Quốc hội ban hành Nghị quyết quy định về cơ chế đặc thù đầu tư, đầu tư công, mua sắm công các sản phẩm, dịch vụ số để thúc đẩy mạnh mẽ chuyển đổi số quốc gia giai đoạn 2025-2030, báo cáo Chính phủ trong tháng 5.2025.    Bố trí ngân sách nhà nước chi cho sự nghiệp khoa học phục vụ các công nghệ chiến lược, đáp ứng yêu cầu Nghị quyết 57-NQ/TW trình cấp có thẩm quyền, bảo đảm bố trí đủ ít nhất 3% ngân sách cho khoa học công nghệ và tiếp tục nâng tỷ lệ lên 2% GDP trong 5 năm tiếp theo.    Có thể bố trí ngay tỷ lệ 5% GDP nếu thấy cần thiết và sử dụng được, tạo niềm tin cho các nhà khoa học, các doanh nghiệp, hoàn thành trong tháng 12.2025.    Tái cấu trúc quy trình, tái sử dụng dữ liệu số hóa để cắt giảm, không yêu cầu người dân, doanh nghiệp phải xuất trình, nộp, khai báo lại các thông tin, giấy tờ đã được số hóa; ưu tiên nghiên cứu đưa vào tái sử dụng dữ liệu đất đai đã được số hóa phục vụ cắt giảm các thủ tục hành chính về cư trú; các đơn vị hành chính cấp huyện, cấp xã đã hoàn thành số hóa phải đưa vào sử dụng ngay trong Quý II/2025.    Nâng cấp, hoàn thiện hạ tầng công nghệ thông tin đáp ứng yêu cầu tại Văn bản số 1552/BTTTT-THH và Văn bản số 708/BTTTT-CATTT của Bộ Thông tin và Truyền thông (nay là Bộ Khoa học và Công nghệ); hoàn thành kết nối giữa Hệ thống thông tin giải quyết thủ tục hành chính cấp bộ, cấp tỉnh với Cơ sở dữ liệu quốc gia về dân cư phục vụ giải quyết thủ tục hành chính, dịch vụ công theo Nghị định số 107/2021/NĐ-CP của Chính phủ, hoàn thành trong năm 2025.    Xây dựng Đề án chuyển đổi số của bộ, ngành có tính chất tương tự như Đề án 06 và bảo đảm kết nối với Đề án 06 với 11 tiện ích, mục tiêu Bộ Công an đã xây dựng, tập trung nguồn lực, chỉ đạo thực hiện dứt điểm và triển khai trong thời gian từ nay đến hết năm 2025; đưa các tiện ích vào sử dụng thường xuyên phục vụ chuyển đổi số; huy động các nguồn lực xã hội, nguồn lực hợp pháp khác bảo đảm kinh phí triển khai thực hiện.    Về phát triển dữ liệu, Thủ tướng Chính phủ yêu cầu phải tập trung hoàn thành rà soát, cập nhật Chiến lược dữ liệu quốc gia, hoàn thành trong Quý II/2025.    Hoàn thành triển khai xây dựng, đưa vào khai thác, sử dụng các cơ sở dữ liệu quốc gia, cơ sở dữ liệu chuyên ngành (trong đó ưu tiên hoàn thiện cơ sở dữ liệu về đất đai, xây dựng, bảo hiểm, tài chính, doanh nghiệp, lao động việc làm, y tế, giáo dục) và kết nối, xác thực với Cơ sở dữ liệu quốc gia về dân cư trong tháng 8.2025, đồng bộ dữ liệu về Trung tâm dữ liệu quốc gia.    Trước đó, ngày 7.3, Thủ tướng Chính phủ Phạm Minh Chính đã chủ trì Hội nghị trực tuyến toàn quốc quán triệt, triển khai thi hành các luật, nghị quyết vừa được thông qua tại Kỳ họp bất thường lần thứ 9, Quốc hội khóa XV.    Tại hội nghị, Thủ tướng Chính phủ cho biết dự kiến dành gần 10.000 tỉ đồng để thực hiện Nghị quyết 57 của Bộ Chính trị về đột phá phát triển khoa học, công nghệ.");
            
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

```JSON
{
  "task": "processing",
  "action": "semantic_chunk",
  "kwargs": {
    "model_name_or_path": String (path to the model),
	"host": String (for example localhost:11434),
    "threshold_percentile": Float (80.0 t0 95.0),
	"window_size": INT (default value is 2),
    }
  }
}
```

