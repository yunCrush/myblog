# 回调

A调用B，B异步处理逻辑，完成后主动调用A的回调方法。

A方法事先往B方法注册一个回调函数，B完成相关逻辑后，触发回调函数。

{% code title="AClass.java" overflow="wrap" %}
```java
public class AClass {
    public static void main(String[] args) throws InterruptedException {
        BClass b = new BClass();
        System.out.println("A调用第三方支付接口B");
        b.process(new ICallback() {
            //回调对象
            @Override public void methodToCallback() {
                System.out.println("第三方支付完成");
            }
        });
        }
}

```
{% endcode %}

{% code title="BClass.java" overflow="wrap" %}
```java
public class BClass {
    public void process(ICallback callback) {
    //...
        System.out.println("第三方支付收到请求，开始处理支付逻辑");
        new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    TimeUnit.SECONDS.sleep(5);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("支付完成，开始异步回调Aclass");
                callback.methodToCallback();
            }
        }).start();
        System.out.println("Bclass 非阻塞直接结束");
    //...
    }
}
```
{% endcode %}

{% code title="ICallback.java" overflow="wrap" %}
```java
public interface ICallback {
    void methodToCallback();
}
```
{% endcode %}

执行结果：

{% code lineNumbers="true" %}
```
调用第三方支付接口
第三方支付收到请求，开始处理支付逻辑
Bclass 非阻塞直接结束
支付完成，开始异步回调Aclass
第三方支付完成
```
{% endcode %}
