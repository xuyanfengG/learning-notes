> SimpleDateFormat是线程不安全的。SimpleDateFormat类内部有一个Calendar对象引用，它用来储存和这个SimpleDateFormat相关的日期信息，例如：**sdf.parse(dateStr),sdf.format(date) **诸如此类的方法参数传入的日期相关String,Date等等，都是交由Calendar引用来储存的.这样就会导致一个问题，如果你的SimpleDateFormat是个static的，那么多个thread 之间就会共享这个SimpleDateFormat，同时也是共享这个Calendar引用。单例、多线程、又有成员变量（这个变量在方法中是可以修改的），这个场景是不是很像servlet，在高并发的情况下，容易出现幻读成员变量的现象，故说SimpleDateFormat是线程不安全的对象。
>
> jdk8及以上版本可以使用Instant代替Date，LocalDateTime代替Calendar，DateTimeFormatter代替Simpledateformatter。

```java
// 高并发下使用ThreadLocal处理
private static final ThreadLocal<DateFormat> format_1 = new ThreadLocal<DateFormat>(){
    @Override
    protected DateFormat initialValue() {
        return new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
    }
};
```

