# CompletableFuture

### FutureTask

```
// 不见不散 阻塞等待，直到拿到结果
public V get() 
// 过时不候
public V get(long timeout, TimeUnit unit)

// 任务是否完成
public boolean isDone() 
```

### CompletableFuture

implement CompletionStage(异步中一个阶段，一个阶段完成以后会触发另一个阶段), Future\
Main线程不应当立刻结束，否则**默认的线程池**会立即关闭

#### **1. runAsync VS supplyAsync**

```
// 创建一个无返回值的异步操作,不传入线程池则使用ForkJoinPool线程池
public static CompletableFuture<Void> runAsync(Runnable runnable) 
// 自带线程池的，同上
public static CompletableFuture<Void> runAsync(Runnable runnable,Executor executor) 


// 创建一个有返回值的异步操作
public static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier) 
// 自带线程池的，同上
public static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier, Executor executor)


// 阻塞等待，直到返回一个值
public T get() 
// 超时则不再等待 
public T get(long timeout, TimeUnit unit)

// 是否完成，未完成则返回false,get()得到默认值value
public boolean complete(T value) 
```

#### **2. thenApply VS handle**

```
//当一个线程依赖另一个线程时，使用thenApply串行化两个线程。如果出现Exception则停止向下运行
public <U> CompletableFuture<U> thenApply(Function<? super T,? extends U> fn) 

//当一个线程依赖另一个线程时，使用handle串行化两个线程。出现Exception,带着exception继续运行
public <U> CompletableFuture<U> handle(BiFunction<? super T, Throwable, ? extends U> fn) 
```

#### **3. whenComplete VS whenCompleteAsync**

```
// Bi表示传入两个参数 执行任务的线程继续执行后续逻辑
public CompletableFuture<T> whenComplete(BiConsumer<? super T, ? super Throwable> action) 

// 交给线程池中的线程执行后序逻辑，可能是当前线程，也可能是其他
public CompletableFuture<T> whenCompleteAsync(BiConsumer<? super T, ? super Throwable> action) 

// 对异常进行处理
public CompletableFuture<T> exceptionally(Function<Throwable, ? extends T> fn) 
```

#### **4. applyToEither VS thenCombine**

```
// 两个线程比较，谁运行更快，先完成任务，则使用谁的结果
public <U> CompletableFuture<U> applyToEither(CompletionStage<? extends T> other, Function<? super T, U> fn) 

// 两个CompletionStage任务都完成后，最终能把两个任务的结果一起交给thenCombine 来处理
public <U,V> CompletableFuture<V> thenCombine(CompletionStage<? extends U> other, BiFunction<? super T,? super U,? extends V> fn) 
```

#### **5. thenAccept VS thenRun**

```
// 接收参数，消费参数后无返回值
 public CompletableFuture<Void> thenAccept(Consumer<? super T> action)

// 执行完上一个任务A后，再执行B, B不需要A作参数
public CompletableFuture<Void> thenRun(Runnable action)

// 接收一个参数，执行任务完后，无返回值
public CompletableFuture<Void> thenAccept(Consumer<? super T> action) 
```

**示例代码:** [completabelFuture](https://github.com/yunCrush/JUC/tree/main/src/main/java/com/yun/completabelFuture)
