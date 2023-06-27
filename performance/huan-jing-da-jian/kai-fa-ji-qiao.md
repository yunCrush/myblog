# 编程技巧

### 1. Optional规避 空指针(NPE)

Optional：最多存放一个对象的集合。

&#x20;    包装类自动拆箱会导致空指针异常，为什么会需要包装类呢？包装类是为了给基本类型赋予对象一样的特性，集合里存放的是对象，正常情况下优先使用基本类型。

```
 Long count = null;
// 使用javac ,javap -c进行反编译查看汇编码是由于调用了Long.longValue()方法
// 自动拆箱： 导致下方的空指针
long coun2 = count;
```

```
 public static Human help(){
        return new Human();
    }
    static class Human {
        String name;
        public String getName(){
            return name;
        }
    }
    
    
public static void main(String[] args) {
        Optional<Human> human = Optional.empty();
        // 不存在就返回一个默认值
        human.orElse(new Human());

        // 不存在返回一个函数构造的默认值
        human.orElseGet( () -help());

        // 不存在就抛出一个异常
        human.orElseThrow(RuntimeException::new);

        // map human 中的对象执行某种操作， 返回的是一个Optional，可以无限级联
        // Human humans = new Human();
        human.map( u -> u.getName()).map(name -> name.length()).map(num -> num > 0);
    
}
```

### 2.格式转换

```
public String obj2Json(PersonEntity person) {
	String json = JSONObject.toJSONString(person);
	log.info("Object=====1=====Json: {}", json);
	return json;
}
```

```
public void json2Obj(String json) {
	PersonEntity personEntity = JSON.parseObject(json, PersonEntity.class);
	log.info("Json=====2=====Person: {}", personEntity.toString());
}
```

```
public JSONArray list2Arr(List<String> list) {
	log.info(JSON.toJSONString(list));
	log.info("List====3=====Array: {}", JSONArray.parseArray(JSON.toJSONString(list)));
	return JSONArray.parseArray(JSON.toJSONString(list));
}
```

```
public List<String> jsonArr2List(String jsonArray) {
	List<String> list = JSON.parseArray(jsonArray,String.class);
	log.info("遍历list:");
	list.forEach((str) -> log.info("{}",str));
	return list;
	}
```

测试方法

```
public static void main(String[] args) {
        JSONTest test = new JSONTest();
        
        List<String> list = new ArrayList();
        list.add("北京");
        list.add("南京");
        PersonEntity person = new PersonEntity("yuncrush", 24, list);
        String json = test.obj2Json(person);
        
        test.json2Obj(json);
        String jsonArr = test.list2Arr(list).toJSONString();
        test.jsonArr2List(jsonArr);

}

@Data
@ToString
@AllArgsConstructor
public class PersonEntity {
	String name;
	int age;
	List<String> address;
}
```
