# Create Vector Database

The purpose of this function is to convert an arrow file data to a vector database, where text is converted into long vectors (embedding vectors).

The arrow file should have at least two columns: `index` and `source_column`. It is OK if you have multiple columns, but these columns [index, source_column] are required.


Java sampel code to call the service:

```JAVA
	public class CreateTable {
		public static void main(String[] args) {
			PythonInterpreterConfig config = PythonInterpreterConfig
				.newBuilder()
				.setPythonExec("python")
				.build();

			try (PythonInterpreter interpreter = new PythonInterpreter(config)) {
				interpreter.exec("from titan.tbrane.services import worker");
				HashMap<String, String> kwargs = new HashMap<>();
				kwargs.put("database_path", "/path/to/database");
				kwargs.put("table_name", "your_table_name");
				kwargs.put("arrow_file", "/path/to/arrow_file");
				kwargs.put("source_column", "text");
				kwargs.put("language", "vi");

				HashMap<String, Object> params = new HashMap<>();
				params.put("task", "processing");
				params.put("action", "create_table_from_arrow");
				params.put("kwargs", kwargs);

				Object result = interpreter.invoke("worker", params);

				if (result instanceof Map<?, ?>) {
					@SuppressWarnings("unchecked")
					Map<String, Object> castedResult = (Map<String, Object>) result;
					System.out.println(castedResult.get("table_object"));
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

Set path to database (String)
kwargs.put("database_path", "/path/to/database");

Set the table name (String)
kwargs.put("table_name", "your_table_name");

Set path to input arrow file (String)
kwargs.put("arrow_file", "/path/to/arrow_file");

Assign value (String) for `source_column`.
kwargs.put("source_column", "text");

Set the language (String), support `vi` and `en`
kwargs.put("language", "vi");
