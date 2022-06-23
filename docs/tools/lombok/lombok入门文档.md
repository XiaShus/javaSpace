## lombok

以前的Java项目中，充斥着太多不友好的代码：POJO的getter/setter/toString；异常处理；I/O流的关闭操作等等，这些样板代码既没有技术含量，又影响着代码的美观，Lombok应运而生。简单来说就是用来干掉一部分重复代码的。



### 为什么推荐使用Lombok

简单，好用，功能强大。因为是编译器生成具体方法的，所以说不会影响执行效率，你可以查看 java 代码编译后的 class 文件。

Book.java

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class Book {
    private String id;
    private String name;
}
```



Book.class

```java
public class Book {
    private String id;
    private String name;

    public String getId() {
        return this.id;
    }

    public String getName() {
        return this.name;
    }

    public void setId(final String id) {
        this.id = id;
    }

    public void setName(final String name) {
        this.name = name;
    }

    public boolean equals(final Object o) {
        if (o == this) {
            return true;
        } else if (!(o instanceof Book)) {
            return false;
        } else {
            Book other = (Book)o;
            if (!other.canEqual(this)) {
                return false;
            } else {
                Object this$id = this.getId();
                Object other$id = other.getId();
                if (this$id == null) {
                    if (other$id != null) {
                        return false;
                    }
                } else if (!this$id.equals(other$id)) {
                    return false;
                }

                Object this$name = this.getName();
                Object other$name = other.getName();
                if (this$name == null) {
                    if (other$name != null) {
                        return false;
                    }
                } else if (!this$name.equals(other$name)) {
                    return false;
                }

                return true;
            }
        }
    }

    protected boolean canEqual(final Object other) {
        return other instanceof Book;
    }

    public int hashCode() {
        int PRIME = true;
        int result = 1;
        Object $id = this.getId();
        int result = result * 59 + ($id == null ? 43 : $id.hashCode());
        Object $name = this.getName();
        result = result * 59 + ($name == null ? 43 : $name.hashCode());
        return result;
    }

    public String toString() {
        return "Book(id=" + this.getId() + ", name=" + this.getName() + ")";
    }

    public Book() {
    }

    public Book(final String id, final String name) {
        this.id = id;
        this.name = name;
    }
}
```



### 常用注解

- **@Data：**为所有字段生成 getter，一个有用的 toString 方法，以及检查所有非瞬态字段的 hashCode 和 equals 实现。还将为所有非最终字段以及构造函数生成设置器。
  等效于@Getter @Setter @RequiredArgsConstructor @ToString @EqualsAndHashCode 
- **@NoArgsConstructor：**生成一个无参数的构造函数。如果由于存在 final 字段而无法编写此类构造函数，将生成错误消息。
- **@AllArgsConstructor：**生成一个全参数构造函数。全参数构造函数要求类中的每个字段都有一个参数。
- **@Getter：**生成该类属性的所有 get 方法。
- **@Setter：**生成该类属性的所有 set方法。
- **@RequiredArgsConstructor：**
- **@ToString：**生成该类的 toString 方法。
- **@EqualsAndHashCode ：**生成该类的 equals 和 hashCode 方法。



