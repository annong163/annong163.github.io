---
title: JDK8—Stream集合操作快速上手
author: AnNong
cover: true
categories: JAVA
summary: Java 8引入了全新的Stream API，stream是对集合对象功能的增强。
tags:
  - JDK8
  - 集合
abbrlink: a20afd81
date: 2022-04-26 15:38:44
---

## Stream简介
- Java 8引入了全新的Stream API。这里的Stream和I/O流不同，它更像具有Iterable的集合类，但行为和集合类又有所不同。
- stream是对集合对象功能的增强，它专注于对集合对象进行各种非常便利、高效的聚合操作，或者大批量数据操作。
- 只要给出需要对其包含的元素执行什么操作，比如 “过滤掉长度大于 10 的字符串”、“获取每个字符串的首字母”等，Stream 会隐式地在内部进行遍历，做出相应的数据转换。

## 为什么要使用Stream
- 函数式编程带来的好处尤为明显。这种代码更多地表达了业务逻辑的意图，而不是它的实现机制。易读的代码也易于维护、更可靠、更不容易出错。
- 高端 (▼へ▼メ)

## 实例数据源
```java
public class Data {
private static List<PersonModel> list = null;

    static {
        PersonModel wu = new PersonModel("wu qi", 18, "男");
        PersonModel zhang = new PersonModel("zhang san", 19, "男");
        PersonModel wang = new PersonModel("wang si", 20, "女");
        PersonModel zhao = new PersonModel("zhao wu", 20, "男");
        PersonModel chen = new PersonModel("chen liu", 21, "男");
        list = Arrays.asList(wu, zhang, wang, zhao, chen);
    }

    public static List<PersonModel> getData() {
        return list;
    }
}
```

## Filter
- 遍历数据并检查其中的元素时使用。
- filter接受一个函数作为参数，该函数用Lambda表达式表示。
```java
    /**
     * 过滤所有的男性
     */
    public static void fiterSex(){
        List<PersonModel> data = Data.getData();

        //old
        List<PersonModel> temp=new ArrayList<>();
        for (PersonModel person:data) {
            if ("男".equals(person.getSex())){
                temp.add(person);
            }
        }
        System.out.println(temp);
        //new
        List<PersonModel> collect = data
                .stream()
                .filter(person -> "男".equals(person.getSex()))
                .collect(toList());
        System.out.println(collect);
    }

    /**
     * 过滤所有的男性 并且小于20岁
     */
    public static void fiterSexAndAge(){
        List<PersonModel> data = Data.getData();

        //old
        List<PersonModel> temp=new ArrayList<>();
        for (PersonModel person:data) {
            if ("男".equals(person.getSex())&&person.getAge()<20){
                temp.add(person);
            }
        }

        //new 1
        List<PersonModel> collect = data
                .stream()
                .filter(person -> {
                    if ("男".equals(person.getSex())&&person.getAge()<20){
                        return true;
                    }
                    return false;
                })
                .collect(toList());
        //new 2
        List<PersonModel> collect1 = data
                .stream()
                .filter(person -> ("男".equals(person.getSex())&&person.getAge()<20))
                .collect(toList());

    }
```

## Map
- map生成的是个一对一映射,for的作用
```javascript
   /**
     * 取出所有的用户名字
     */
    public static void getUserNameList(){
        List<PersonModel> data = Data.getData();

        //old
        List<String> list=new ArrayList<>();
        for (PersonModel persion:data) {
            list.add(persion.getName());
        }
        System.out.println(list);

        //new 1
        List<String> collect = data.stream().map(person -> person.getName()).collect(toList());
        System.out.println(collect);

        //new 2
        List<String> collect1 = data.stream().map(PersonModel::getName).collect(toList());
        System.out.println(collect1);

        //new 3
        List<String> collect2 = data.stream().map(person -> {
            System.out.println(person.getName());
            return person.getName();
        }).collect(toList());
    }
```

## Reduce
- 感觉类似递归
- 数字(字符串)累加
```javascript
 public static void reduceTest(){
        //累加，初始化值是 10
        Integer reduce = Stream.of(1, 2, 3, 4)
                .reduce(10, (count, item) ->{
            System.out.println("count:"+count);
            System.out.println("item:"+item);
            return count + item;
        } );
        System.out.println(reduce);

        Integer reduce1 = Stream.of(1, 2, 3, 4)
                .reduce(0, (x, y) -> x + y);
        System.out.println(reduce1);

        String reduce2 = Stream.of("1", "2", "3")
                .reduce("0", (x, y) -> (x + "," + y));
        System.out.println(reduce2);
    }
```

## Collect
- collect在流中生成列表，map，等常用的数据结构
- toList()
- toSet()
- toMap()
- 自定义
```javascript
 /**
     * toList
     */
    public static void toListTest(){
        List<PersonModel> data = Data.getData();
        List<String> collect = data.stream()
                .map(PersonModel::getName)
                .collect(Collectors.toList());
    }

    /**
     * toSet
     */
    public static void toSetTest(){
        List<PersonModel> data = Data.getData();
        Set<String> collect = data.stream()
                .map(PersonModel::getName)
                .collect(Collectors.toSet());
    }

    /**
     * toMap
     */
    public static void toMapTest(){
        List<PersonModel> data = Data.getData();
        Map<String, Integer> collect = data.stream()
                .collect(
                        Collectors.toMap(PersonModel::getName, PersonModel::getAge)
                );

        data.stream()
                .collect(Collectors.toMap(per->per.getName(), value->{
            return value+"1";
        }));
    }

    /**
     * 指定类型
     */
    public static void toTreeSetTest(){
        List<PersonModel> data = Data.getData();
        TreeSet<PersonModel> collect = data.stream()
                .collect(Collectors.toCollection(TreeSet::new));
        System.out.println(collect);
    }

    /**
     * 分组
     */
    public static void toGroupTest(){
        List<PersonModel> data = Data.getData();
        Map<Boolean, List<PersonModel>> collect = data.stream()
                .collect(Collectors.groupingBy(per -> "男".equals(per.getSex())));
        System.out.println(collect);
    }

    /**
     * 分隔
     */
    public static void toJoiningTest(){
        List<PersonModel> data = Data.getData();
        String collect = data.stream()
                .map(personModel -> personModel.getName())
                .collect(Collectors.joining(",", "{", "}"));
        System.out.println(collect);
    }

    /**
     * 自定义
     */
    public static void reduce(){
        List<String> collect = Stream.of("1", "2", "3").collect(
                Collectors.reducing(new ArrayList<String>(), x -> Arrays.asList(x), (y, z) -> {
                    y.addAll(z);
                    return y;
                }));
        System.out.println(collect);
    }
```

## Optional
- Optional 是为核心类库新设计的一个数据类型，用来替换 null 值。
- 人们对原有的 null 值有很多抱怨，甚至连发明这一概念的Tony Hoare也是如此，他曾说这是自己的一个“价值连城的错误”
- 用处很广，不光在lambda中，哪都能用
- Optional.of(T)，T为非空，否则初始化报错
- Optional.ofNullable(T)，T为任意，可以为空
- isPresent()，相当于 ！=null
- ifPresent(T)， T可以是一段lambda表达式 ，或者其他代码，非空则执行

```javascript
public static void main(String[] args) {


        PersonModel personModel=new PersonModel();

        //对象为空则打出 -
        Optional<Object> o = Optional.of(personModel);
        System.out.println(o.isPresent()?o.get():"-");

        //名称为空则打出 -
        Optional<String> name = Optional.ofNullable(personModel.getName());
        System.out.println(name.isPresent()?name.get():"-");

        //如果不为空，则打出xxx
        Optional.ofNullable("test").ifPresent(na->{
            System.out.println(na+"ifPresent");
        });

        //如果空，则返回指定字符串
        System.out.println(Optional.ofNullable(null).orElse("-"));
        System.out.println(Optional.ofNullable("1").orElse("-"));

        //如果空，则返回 指定方法，或者代码
        System.out.println(Optional.ofNullable(null).orElseGet(()->{
            return "hahah";
        }));
        System.out.println(Optional.ofNullable("1").orElseGet(()->{
            return "hahah";
        }));

        //如果空，则可以抛出异常
        System.out.println(Optional.ofNullable("1").orElseThrow(()->{
            throw new RuntimeException("ss");
        }));


//        Objects.requireNonNull(null,"is null");


        //利用 Optional 进行多级判断
        EarthModel earthModel1 = new EarthModel();
        //old
        if (earthModel1!=null){
            if (earthModel1.getTea()!=null){
                //...
            }
        }
        //new
        Optional.ofNullable(earthModel1)
                .map(EarthModel::getTea)
                .map(TeaModel::getType)
                .isPresent();


//        Optional<EarthModel> earthModel = Optional.ofNullable(new EarthModel());
//        Optional<List<PersonModel>> personModels = earthModel.map(EarthModel::getPersonModels);
//        Optional<Stream<String>> stringStream = personModels.map(per -> per.stream().map(PersonModel::getName));


        //判断对象中的list
        Optional.ofNullable(new EarthModel())
                .map(EarthModel::getPersonModels)
                .map(pers->pers
                        .stream()
                        .map(PersonModel::getName)
                        .collect(toList()))
                .ifPresent(per-> System.out.println(per));


        List<PersonModel> models=Data.getData();
        Optional.ofNullable(models)
                .map(per -> per
                        .stream()
                        .map(PersonModel::getName)
                        .collect(toList()))
                .ifPresent(per-> System.out.println(per));

    }
```
## 并发
- stream替换成parallelStream或 parallel
- 输入流的大小并不是决定并行化是否会带来速度提升的唯一因素，性能还会受到编写代码的方式和核的数量的影响
- 影响性能的五要素是:数据大小、源数据结构、值是否装箱、可用的CPU核数量，以及处理每个元素所花的时间
```javascript
 //根据数字的大小，有不同的结果
    private static int size=10000000;
    public static void main(String[] args) {
        System.out.println("-----------List-----------");
        testList();
        System.out.println("-----------Set-----------");
        testSet();
    }

    /**
     * 测试list
     */
    public static void testList(){
        List<Integer> list = new ArrayList<>(size);
        for (Integer i = 0; i < size; i++) {
            list.add(new Integer(i));
        }

        List<Integer> temp1 = new ArrayList<>(size);
        //老的
        long start=System.currentTimeMillis();
        for (Integer i: list) {
            temp1.add(i);
        }
        System.out.println(+System.currentTimeMillis()-start);

        //同步
        long start1=System.currentTimeMillis();
        list.stream().collect(Collectors.toList());
        System.out.println(System.currentTimeMillis()-start1);

        //并发
        long start2=System.currentTimeMillis();
        list.parallelStream().collect(Collectors.toList());
        System.out.println(System.currentTimeMillis()-start2);
    }

    /**
     * 测试set
     */
    public static void testSet(){
        List<Integer> list = new ArrayList<>(size);
        for (Integer i = 0; i < size; i++) {
            list.add(new Integer(i));
        }

        Set<Integer> temp1 = new HashSet<>(size);
        //老的
        long start=System.currentTimeMillis();
        for (Integer i: list) {
            temp1.add(i);
        }
        System.out.println(+System.currentTimeMillis()-start);

        //同步
        long start1=System.currentTimeMillis();
        list.stream().collect(Collectors.toSet());
        System.out.println(System.currentTimeMillis()-start1);

        //并发
        long start2=System.currentTimeMillis();
        list.parallelStream().collect(Collectors.toSet());
        System.out.println(System.currentTimeMillis()-start2);
    }
```


