# Query RAG VectorDB

Once the vector database is created, we now can query the database to get the result for RAG model.


Java sampel code to call the service:

```JAVA
	public class LoadTableQuery {
		public static void main(String[] args) {
			PythonInterpreterConfig config = PythonInterpreterConfig
				.newBuilder()
				.setPythonExec("python")
				.build();

			try (PythonInterpreter interpreter = new PythonInterpreter(config)) {
				interpreter.exec("from titan.tbrane.services import worker");

				HashMap<String, String> kwargs1 = new HashMap<>();
				kwargs1.put("database_path", "/path/to/database");
				kwargs1.put("table_name", "your_table_name");

				HashMap<String, Object> params1 = new HashMap<>();
				params1.put("task", "processing");
				params1.put("action", "load_table");
				params1.put("kwargs", kwargs1);

				long startTime = System.currentTimeMillis();

				Object result = interpreter.invoke("worker", params1);

				long endTime = System.currentTimeMillis();

				long elapsedTime = endTime - startTime;

				System.out.println("Loading table: " + elapsedTime + " milliseconds");

				@SuppressWarnings("unchecked")
				Map<String, Object> castedResult = (Map<String, Object>) result;

				Object table_obj = castedResult.get("table_object");
				System.out.println(table_obj);

				HashMap<String, Object> kwargs2 = new HashMap<>();
				kwargs2.put("table", table_obj);
				kwargs2.put("limit", 3);

				HashMap<String, Object> params2 = new HashMap<>();
				params2.put("task", "querying");
				params2.put("action", "query_from_table");
				params2.put("kwargs", kwargs2);

				Object result2;

				for (int i = 1; i <= 3; i++) {
					kwargs2.put("query_text", "Cho tôi biết tình hình lợi nhuận công ty quý vừa qua");

					startTime = System.currentTimeMillis();

					result2 = interpreter.invoke("worker", params2);

					endTime = System.currentTimeMillis();

					elapsedTime = endTime - startTime;

					System.out.println("Query time for i= " + i + " in " + elapsedTime + " milliseconds");
					System.out.println(result2);

				}

			} catch (Exception e) {
				e.printStackTrace();
			}
		}
	}
```


The structure of the params HashMap

"task": (String) must be an existing task. Currently has: `processing` ; `querying` ; `reforming`

"action": (String) must be supported actions. Supported: `create_table_from_arrow`; `load_table`; `add_arrow_to_table`

"kwargs": another HashMap to pass the arguments to action function.

**Notice:** The first time query always takes longer than any query from the second time.
