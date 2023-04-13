# 1.Singleton-单例模式

<font color=OrangeRed size=4><b>单例模式有8中写法</b></font>


<table><tr><td bgcolor=DeepSkyBlue><font size=5><b>第一种写法</td></tr></table>

```java
package cn.yiyuyyds.singleton;


/**
 * 饿汉式
 * 类加载到内存后，就实例化一个单例，JVM保证线程安全
 * 简单实用，推荐使用
 * 唯一缺点，不管有没有用到，类加载时就完成实例化
 */
public class Mgr01 {
    private static final Mgr01 INSTANCE = new Mgr01();

    private Mgr01() {
    }

    public static Mgr01 getInstance() {
        return INSTANCE;
    }

    public void m() {
        System.out.println("m");
    }

    public static void main(String[] args) {
        Mgr01 m1 = Mgr01.getInstance();
        Mgr01 m2 = Mgr01.getInstance();
        System.out.println(m1 == m2);
    }
}
```


<table><tr><td bgcolor=DeepSkyBlue><font size=5><b>第二种写法</td></tr></table>


```java
package cn.yiyuyyds.singleton;


/**
 * 和01是一个意思
 */
public class Mgr02 {
    private static final Mgr02 INSTANCE;

    static {
        INSTANCE = new Mgr02();
    }

    private Mgr02() {
    }

    public static Mgr02 getInstance() {
        return INSTANCE;
    }

    public void m() {
        System.out.println("m");
    }

    public static void main(String[] args) {
        Mgr02 m1 = Mgr02.getInstance();
        Mgr02 m2 = Mgr02.getInstance();
        System.out.println(m1 == m2);
    }
}
```



<table><tr><td bgcolor=DeepSkyBlue><font size=5><b>第三种写法</td></tr></table>


```java
package cn.yiyuyyds.singleton;


/**
 * lazy loading
 * 也称懒汉式
 * 虽然达到了按需初始化的目的，但却带来线程不安全的问题
 */
public class Mgr03 {
    private static Mgr03 INSTANCE;

    private Mgr03() {
    }

    public static Mgr03 getInstance() {
        if (INSTANCE == null) {
            try {
                Thread.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            INSTANCE = new Mgr03();
        }
        return INSTANCE;
    }

    public void m() {
        System.out.println("m");
    }

    //模拟线程不安全的问题
    public static void main(String[] args) {
        for (int i = 0; i < 100; i++) {
            new Thread(() ->
                    System.out.println(Mgr03.getInstance().hashCode())
            ).start();
        }
    }
}
```

<table><tr><td bgcolor=DeepSkyBlue><font size=5><b>第四种写法</td></tr></table>

```java
package cn.yiyuyyds.singleton;


/**
 * 虽然达到了按需初始化的目的，但却带来线程不安全的问题
 * 可以通过synchronized解决，但也带来效率下降
 */
public class Mgr04 {
    private static Mgr04 INSTANCE;

    private Mgr04() {
    }
    
    public static synchronized Mgr04 getInstance() {
        if (INSTANCE == null) {
            try {
                Thread.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            INSTANCE = new Mgr04();
        }
        return INSTANCE;
    }

    public void m() {
        System.out.println("m");
    }

    public static void main(String[] args) {
        for (int i = 0; i < 100; i++) {
            new Thread(() ->
                    System.out.println(Mgr04.getInstance().hashCode())
            ).start();
        }
    }
}
```


<table><tr><td bgcolor=DeepSkyBlue><font size=5><b>第五种写法</td></tr></table>

```java
package cn.yiyuyyds.singleton;

/**
 * 虽然达到了按需初始化的目的，但却带来线程不安全的问题
 * 可以通过synchronized解决，但也带来效率下降
 */
public class Mgr05 {
    private static Mgr05 INSTANCE;

    private Mgr05() {
    }

    public static Mgr05 getInstance() {
        if (INSTANCE == null) {
            //通过减小同步代码块的方式提高效率，但这种方式还是线程不安全的
            synchronized (Mgr05.class) {
                try {
                    Thread.sleep(1);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                INSTANCE = new Mgr05();
            }
        }
        return INSTANCE;
    }

    public void m() {
        System.out.println("m");
    }

    public static void main(String[] args) {
        for (int i = 0; i < 100; i++) {
            new Thread(() ->
                    System.out.println(Mgr05.getInstance().hashCode())
            ).start();
        }
    }
}
```


<table><tr><td bgcolor=DeepSkyBlue><font size=5><b>第六种写法</td></tr></table>

```java
package cn.yiyuyyds.singleton;


public class Mgr06 {
    private static Mgr06 INSTANCE;

    private Mgr06() {
    }

    public static Mgr06 getInstance() {
        if (INSTANCE == null) {
            //双重检查锁
            synchronized (Mgr06.class) {
                if (INSTANCE == null) {
                    try {
                        Thread.sleep(1);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    INSTANCE = new Mgr06();
                }
            }
        }
        return INSTANCE;
    }

    public void m() {
        System.out.println("m");
    }

    public static void main(String[] args) {
        for (int i = 0; i < 100; i++) {
            new Thread(() ->
                    System.out.println(Mgr06.getInstance().hashCode())
            ).start();
        }
    }
}
```
<font color=Red size=4>补充：private static volatile Mgr06 INSTANCE;</font>


<table><tr><td bgcolor=DeepSkyBlue><font size=5><b>第七种写法</td></tr></table>

```java
package cn.yiyuyyds.singleton;

/**
 * 静态内部类方式
 * JVM保证单例
 * 加载外部类时不会加载内部类，这样可以实现懒加载
 */
public class Mgr07 {
    private Mgr07() {

    }

    private static class Mgr07Holder {
        private static final Mgr07 INSTANCE = new Mgr07();
    }

    public static Mgr07 getInstance() {
        return Mgr07Holder.INSTANCE;
    }

    public void m() {
        System.out.println("m");
    }

    public static void main(String[] args) {
        for (int i = 0; i < 100; i++) {
            new Thread(() ->
                    System.out.println(Mgr07.getInstance().hashCode())
            ).start();
        }
    }
}
```


<table><tr><td bgcolor=DeepSkyBlue><font size=5><b>第八种写法</td></tr></table>

```java
package cn.yiyuyyds.singleton;

/**
 * 不仅可以解决线程同步，还可以防止反序列化
 */
public enum Mgr08 {
    INSTANCE;

    public void m() {
    }

    public static void main(String[] args) {
        for (int i = 0; i < 100; i++) {
            new Thread(() ->
                    System.out.println(Mgr08.INSTANCE.hashCode())
            ).start();
        }
    }
}
```