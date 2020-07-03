一、工厂模式
场景：以牛奶为例，有蒙牛、伊利、爱慕希等品牌。

public interface Milk {

    /**
     * 获得一种牛奶产品
     * @return
     */
    String getName();
}
产品实现接口：

public class Menniu implements Milk{
    @Override
    public String getName() {
        return "蒙牛";
    }
}
public class Aimuxi implements Milk{

    @Override
    public String getName() {
        return "爱慕希";
    }
}
public class Yili implements Milk{
    @Override
    public String getName() {
        return "伊利";
    }
}
1、简单工厂

又叫做静态工厂方法（StaticFactory Method）模式，它的实质是有一个工厂类根据传入的参数，动态的决定应该创建哪一个产品类。

/**
 * 简单工厂（小作坊式）
 * Created by zwz on 2018/8/20.
 */
public class SimpleFactory {
    public Milk getMilk(String name){
        if("爱慕希".equals(name)){
            return new Aimuxi();
        }else if("蒙牛".equals(name)){
            return new Menniu();
        }else if("伊利".equals(name)){
            return new Yili();
        }else{
            System.out.println("不能生产你所需的产品");
            return null;
        }
    }
}
测试类：

public class Test {
    public static void main(String[] args) {
        
        SimpleFactory factory = new SimpleFactory();
        Milk milk = factory.getMilk("爱慕希");
        System.out.println(milk);
    }
}
特点：

从一个方法中获取多中对象。

缺点：

拓展性差，需要创建新的对象时，需要去修改工厂类方法，还需维护标识。



2、工厂方法

工厂方法提供一个工厂接口，让其实现类决定实例化那种产品，并创建对应的类，

首先创个工厂模型：


/**
 * 工厂模型
 * Created by zwz on 2018/8/20.
 */
public interface Factory {
    Milk getMilk();
}
多个工厂实现接口，每个工厂生产不同的产品：


public class AimuxiFactory implements Factory{
    @Override
    public Milk getMilk() {
        return new Aimuxi();
    }
}
public class MenniuFactory implements Factory{
    @Override
    public Milk getMilk() {
        return new Menniu();
    }
}
public class YiliFactory implements Factory {
    @Override
    public Milk getMilk() {
        return new Yili();
    }
}
测试类：

public class FactoryTest {
    public static void main(String[] args) {
        Factory factory = new MenniuFactory();
        System.out.println(factory.getMilk());
    }
}
优点：

一定程度上解耦，消费者不需关心产品实现类如何改变。
一定程度上增加拓展性，若想要拓展产品，只需增加产品实现。
一定程度上增加了代码的封装性，可读性。
缺点：

新增一个产品就需增加一个实现类，会造成代码泛滥。
工厂的创建对用户而言较麻烦。
3、抽象工厂

抽象工厂，是spring中用的最为广泛的一种设计模式

首先创建一个抽象工厂类：

/**
 * 抽象工厂
 * spring中用的最为广泛的一种设计模式
 * 为什么不使用接口：抽象类可以存储一些公共的逻辑，方便统一管理，易于拓展
 * Created by zwz on 2018/8/20.
 */
public abstract class AbstractFactory {

    //这些是公共的逻辑，便于管理

    abstract Milk getMenniu();

    abstract Milk getYili();

    abstract Milk getAimuxi();

}
实现类：

public class MilkFactory extends AbstractFactory{
    @Override
    Milk getMenniu() {
        return new MenniuFactory().getMilk();
    }

    @Override
    Milk getYili() {
        return new YiliFactory().getMilk();
    }

    @Override
    Milk getAimuxi() {
        return new AimuxiFactory().getMilk();
    }
}
测试类：

public class AbstractFactoryTest {
    public static void main(String[] args) {
        MilkFactory factory = new MilkFactory();
        //对用户而言，获取产品更加简单
        System.out.println(factory.getYili());
    }
}
优点：

代码结构简单。
获取产品的过程更加简单。
满足了开闭原则，即对拓展开放，对修改关闭。
缺点：

拓展较繁琐，要拓展时，需同时改动抽象工厂和工厂实现类。