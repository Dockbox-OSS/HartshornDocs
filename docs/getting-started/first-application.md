
??? info "View imports"

    ```java
    import org.dockbox.hartshorn.core.annotations.activate.Activator;
    import org.dockbox.hartshorn.core.boot.HartshornApplication;
    ```
```java
@Activator
public class MyApplication {

    public static void main(String[] args) {
        HartshornApplication.create(MyApplication.class);
    }
}
```