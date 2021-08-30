#### 1.实现方差

```java
import io.confluent.ksql.function.udaf.TableUdaf;
import io.confluent.ksql.function.udaf.UdafDescription;
import io.confluent.ksql.function.udaf.UdafFactory;
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;
import org.apache.kafka.connect.data.Schema;
import org.apache.kafka.connect.data.SchemaBuilder;
import org.apache.kafka.connect.data.Struct;

@UdafDescription(name = "variance", description = "The Variance")
public class Variance {
  private static final String COUNT = "COUNT";
  
  private static final String SUM = "SUM";
  
  private static final String LISTS = "LISTS";
  
  @UdafFactory(description = "The Variance for column with type double", aggregateSchema = "STRUCT<SUM double, COUNT bigint, LISTS string>")
  public static TableUdaf<Double, Struct, Double> Varia() {
    final Schema STRUCT_DOUBLE = SchemaBuilder.struct().optional().field("SUM", Schema.OPTIONAL_FLOAT64_SCHEMA).field("COUNT", Schema.OPTIONAL_INT64_SCHEMA).field("LISTS", Schema.OPTIONAL_STRING_SCHEMA).build();
    return new TableUdaf<Double, Struct, Double>() {
        public Struct undo(Double valueToUndo, Struct aggregate) {
          ArrayList<String> t = Variance.StringToList(aggregate.getString("LISTS"));
          if (t.size() != 0)
            t.remove(t.size() - 1); 
          String t_ = Variance.ListToString(t);
          return (new Struct(STRUCT_DOUBLE))
            .put("SUM", Double.valueOf(aggregate.getFloat64("SUM").doubleValue() - valueToUndo.doubleValue()))
            .put("COUNT", Long.valueOf(aggregate.getInt64("COUNT").longValue() - 1L))
            .put("LISTS", t_);
        }
        
        public Struct initialize() {
          return (new Struct(STRUCT_DOUBLE)).put("SUM", Double.valueOf(0.0D)).put("COUNT", Long.valueOf(0L)).put("LISTS", "");
        }
        
        public Struct aggregate(Double newValue, Struct aggregate) {
          if (newValue == null)
            return aggregate; 
          ArrayList<String> t = Variance.StringToList(aggregate.getString("LISTS"));
          t.add(String.valueOf(newValue));
          String t_ = Variance.ListToString(t);
          return (new Struct(STRUCT_DOUBLE))
            .put("SUM", Double.valueOf(aggregate.getFloat64("SUM").doubleValue() + newValue.doubleValue()))
            .put("COUNT", Long.valueOf(aggregate.getInt64("COUNT").longValue() + 1L))
            .put("LISTS", t_);
        }
        
        public Struct merge(Struct agg1, Struct agg2) {
          ArrayList<String> list1 = Variance.StringToList(agg1.getString("LISTS"));
          ArrayList<String> list2 = Variance.StringToList(agg2.getString("LISTS"));
          return (new Struct(STRUCT_DOUBLE))
            .put("SUM", Double.valueOf(agg1.getFloat64("SUM").doubleValue() + agg2.getFloat64("SUM").doubleValue()))
            .put("COUNT", Long.valueOf(agg1.getInt64("COUNT").longValue() + agg2.getInt64("COUNT").longValue()))
            .put("LISTS", Boolean.valueOf(list1.addAll(list2)));
        }
        
        public Double map(Struct aggregate) {
          long count = aggregate.getInt64("COUNT").longValue();
          if (count == 0L)
            return Double.valueOf(0.0D); 
          double res = 0.0D;
          ArrayList<String> a = Variance.StringToList(aggregate.getString("LISTS"));
          double data = 0.0D;
          double sum = aggregate.getFloat64("SUM").doubleValue();
          for (String d : a) {
            if (!"".equals(d)) {
              data = Double.parseDouble(d);
              res += (data - sum / count) * (data - sum / count);
            } 
          } 
          return Double.valueOf(res / count);
        }
      };
  }
  
  private static String ListToString(List<String> list) {
    if (list == null)
      return ""; 
    return String.join(",", (Iterable)list);
  }
  
  private static ArrayList<String> StringToList(String str) {
    List<String> b = Arrays.asList(str.split(","));
    ArrayList<String> arrayList = new ArrayList<>(b);
    return arrayList;
  }
}
```

