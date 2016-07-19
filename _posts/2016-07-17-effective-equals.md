---
layout: post
title:  "Effective Java之对象通用方法 (57-65)"
date:   2016-07-17 1:05:00
catalog:  true
tags:
    - Effective
    - equals
    - hashcode
    - toString
    - clone
    - Comparable
   
---

# 八、覆盖equals时请遵守通用约定

 对于Object类中提供的equals方法在必要的时候是必要重载的，然而如果违背了一些通用的重载准则，将会给程序带来一些潜在的运行时错误。如果自定义的class没有重载该方法，那么该类实例之间的相等性的比较将是基于两个对象是否指向同一地址来判定的。因此对于以下几种情况可以考虑不重载该方法：
 
      1.    类的每一个实例本质上都是唯一的。
      不同于值对象，需要根据其内容作出一定的判定，然而该类型的类，其实例的自身便具备了一定的唯一性，如Thread、Timer等，他本身并不具备更多逻辑比较的必要性。
      
      2.    不关心类是否提供了“逻辑相等”的测试功能。
      如Random类，开发者在使用过程中并不关心两个Random对象是否可以生成同样随机数的值，对于一些工具类亦是如此，如NumberFormat和DateFormat等。
      
      3.    超类已经覆盖了equals，从超类继承过来的行为对于子类也是合适的。
      如Set实现都从AbstractSet中继承了equals实现，因此其子类将不在需要重新定义该方法，当然这也是充分利用了继承的一个优势。
      
      4.    类是私有的或是包级别私有的，可以确定它的equals方法永远不会被调用。
    
   那么什么时候应该覆盖Object.equals呢？如果类具有自己特有的“逻辑相等”概念，而且超类中没有覆盖equals以实现期望的行为，这是我们就需要覆盖equals方法，如各种值对象，或者像Integer和Date这种表示某个值的对象。在重载之后，当对象插入Map和Set等容器中时，可以得到预期的行为。枚举也可以被视为值对象，然而却是这种情形的一个例外，对于枚举是没有必要重载equals方法，直接比较对象地址即可，而且效率也更高。
   
在覆盖equals是，该条目给出了通用的重载原则：

   1. 自反性：对于非null的引用值x，x.equals(x)返回true。
      如果违反了该原则，当x对象实例被存入集合之后，下次希望从该集合中取出该对象时，集合的contains方法将直接无法找到之前存入的对象实例。
      
   2. 对称性：对于任何非null的引用值x和y，如果y.equals(x)为true，那么x.equals(y)也为true。
   
   
        public final class CaseInsensitiveString {
        private final String s;
        public CaseInsensitiveString(String s) {
            this.s = s;
        }
        @Override public boolean equals(Object o) {
            if (o instanceof CaseInsensitiveString) 
                return s.equalsIgnoreCase((CaseInsensitiveString)o).s);
            if (o instanceof String) //One-way interoperability
                return s.equalsIgnoreCase((String)o);
            return false;
        }
        }
        public static void main(String[] args) {
        CaseInsensitiveString cis = new CaseInsensitiveString("Polish");
        String s = "polish";
        List<CaseInsensitiveString> l = new ArrayList<CaseInsensitiveString>();
        l.add(cis);
        if (l.contains(s)) 
            System.out.println("s can be found in the List");
        }
        
 对于上例，如果执行cis.equals(s)将会返回true，因为在该class的equals方法中对参数o的类型针对String作了特殊的判断和特殊的处理，因此如果equals中传入的参数类型为String时，可以进一步完成大小写不敏感的比较。然而在String的equals中，并没有针对CaseInsensitiveString类型做任何处理，因此s.equals(cis)将一定返回false。针对该示例代码，由于无法确定List.contains的实现是基于**cis.equals(s)还是基于s.equals(cis)**，对于实现逻辑两者都是可以接受的，既然如此，外部的使用者在调用该方法时也应该同样保证并不依赖于底层的具体实现逻辑。由此可见，equals方法的对称性是非常必要的。以上的equals实现可以做如下修改：

     @Override public boolean equals(Object o) {
        if (o instanceof CaseInsensitiveString) 
            return s.equalsIgnoreCase((CaseInsensitiveString)o).s);
        return false;
    }

这样修改之后，cis.equals(s)和s.equals(cis)都将返回false。  

3. 传递性：对于任何非null的引用值x、y和z，如果x.equals(y)返回true，同时y.equals(z)也返回true，那么x.equals(z)也必须返回true。

        public class Point {
          private final int x;
          private final int y;
          public Point(int x,int y) {
            this.x = x;
            this.y = y;
          }
          @Override public boolean equals(Object o) {
            if (!(o instanceof Point)) 
                return false;
            Point p = (Point)o;
            return p.x == x && p.y == y;
          }
        }
        
对于该类的equals重载是没有任何问题了，该逻辑可以保证传递性，然而在我们试图给Point类添加新的子类时，会是什么样呢？

      public class ColorPoint extends Point {
        private final Color c;
        public ColorPoint(int x,int y,Color c) {
            super(x,y);
            this.c = c;
        }
        @Override public boolean equals(Object o) {
            if (!(o instanceof ColorPoint)) 
                return false;
            return super.equals(o) && ((ColorPoint)o).c == c;
        }
    }
    
如果在ColorPoint中没有重载自己的equals方法而是直接继承自超类，这样的相等性比较逻辑将会给使用者带来极大的迷惑，毕竟Color域字段对于ColorPoint而言确实是非常有意义的比较性字段，因此该类重载了自己的equals方法。然而这样的重载方式确实带来了一些潜在的问题，见如下代码：

     public void test() {
        Point p = new Point(1,2);
        ColorPoint cp = new ColorPoint(1,2,Color.RED);
        if (p.equals(cp))
            System.out.println("p.equals(cp) is true");
        if (!cp.equals(p))
            System.out.println("cp.equals(p) is false");
    }
    
从输出结果来看，ColorPoint.equals方法破坏了相等性规则中的对称性，因此需要做如下修改：

    @Override public boolean equals(Object o) {
        if (!(o instanceof Point)) 
            return false;
        if (!(o instanceof ColorPoint))
            return o.equals(this);
        return super.equals(o) && ((ColorPoint)o).c == c;
    }
    
经过这样的修改，对称性确实得到了保证，但是却牺牲了传递性，见如下代码：

    public void test() {
        ColorPoint p1 = new ColorPoint(1,2,Color.RED);
        Point p2 = new Point(1,2);
        ColorPoint p1 = new ColorPoint(1,2,Color.BLUE);
        if (p1.equals(p2) && p2.equals(p3))
            System.out.println("p1.equals(p2) && p2.equals(p3) is true");
        if (!(p1.equals(p3))
            System.out.println("p1.equals(p3) is false");
    }
    
再次看输出结果，传递性确实被打破了。如果我们在Point.equals中不使用instanceof而是直接使用getClass呢？

    @Override public boolean equals(Object o) {
         if (o == null || o.getClass() == getClass()) 
             return false;
         Point p = (Point)o;
         return p.x == x && p.y == y;
     }
     
 这样的Point.equals确实保证了对象相等性的这几条规则，然而在实际应用中又是什么样子呢？
 
     class MyTest {
        private static final Set<Point> unitCircle;
        static {
            unitCircle = new HashSet<Point>();
            unitCircle.add(new Point(1,0));
            unitCircle.add(new Point(0,1));
            unitCircle.add(new Point(-1,0));
            unitCircle.add(new Point(0,-1));
        }
        public static boolean onUnitCircle(Point p) {
            return unitCircle.contains(p);
        }
    }
    
 如果此时我们测试的不是Point类本身，而是ColorPoint，那么按照目前Point.equals(getClass方式)的实现逻辑，ColorPoint对象在被传入onUnitCircle方法后，将永远不会返回true，这样的行为违反了"里氏替换原则"(敏捷软件开发一书中给出了很多的解释)，既一个类型的任何重要属性也将适用于它的子类型。因此该类型编写的任何方法，在它的子类型上也应该同样运行的很好。
 
   如何解决这个问题，该条目给出了一个折中的方案，既复合优先于继承，见如下代码：
   
     public class ColorPoint {
        //包含了Point的代理类
        private final Point p;
        private final Color c;
        public ColorPoint(int x,int y,Color c) {
            if (c == null)
                throw new NullPointerException();
            p = new Point(x,y);
            this.c = c;
        }
        //提供一个视图方法返回内部的Point对象实例。这里Point实例为final对象非常重要，
    //可以避免使用者的误改动。视图方法在Java的集合框架中有着大量的应用。
        public Point asPoint() {
            return p;
        }
        @Override public boolean equals(Object o) {
            if (!(o instanceof ColorPoint)) 
                return false;
            ColorPoint cp = (ColorPoint)o;
            return cp.p.equals(p) && cp.c.equals(c);
        }
    }
    
    
4.    一致性：对于任何非null的引用值x和y，只要equals的比较操作在对象中所用的信息没有被改变，多次调用x.equals(y)就会一致的返回true，或者一致返回false。
      在实际的编码中，尽量不要让类的equals方法依赖一些不确定性较强的域字段，如path。由于path有多种表示方式可以指向相同的目录，特别是当path中包含主机名称或ip地址等信息时，更增加了它的不确定性。再有就是path还存在一定的平台依赖性。
      
 5. 非空性：很难想象会存在o.equals(null)返回true的正常逻辑。作为JDK框架中极为重要的方法之一，equals方法被JDK中的基础类广泛的使用，因此作为一种通用的约定，像equals、toString、hashCode和compareTo等重要的通用方法，开发者在重载时不应该让自己的实现抛出异常，否则会引起很多潜在的Bug。如在Map集合中查找指定的键，由于查找过程中的键相等性的比较就是利用键对象的equals方法，如果此时重载后的equals方法抛出NullPointerException异常，而Map的get方法并未捕获该异常，从而导致系统的运行时崩溃错误，然而事实上，这样的问题是完全可以通过正常的校验手段来避免的。综上所述，很多对象在重载equals方法时都会首先对输入的参数进行是否为null的判断，见如下代码：
 
        @Override public boolean equals(Object o) {
        if (o == null)
            return false;
        if (!(o instanceof MyType)) 
            return false;
        ...
        }
        
  注意以上代码中的instanceof判断，由于在后面的实现中需要将参数o进行类型强转，如果类型不匹配则会抛出ClassCastException，导致equals方法提前退出。在此需要指出的是instanceof还有一个潜在的规则，**如果其左值为null，instanceof操作符将始终返回false**(null不是Object的子类)，因此上面的代码可以优化为：
  
       @Override public boolean equals(Object o) {
         if (!(o instanceof MyType)) 
             return false;
         ...
     }
     
 鉴于之上所述，该条目中给出了重载equals方法的最佳逻辑：
 
  1.    使用==操作符检查"参数是否为这个对象的引用"，如果是则返回true。由于==操作符是基于对象地址的比较，因此特别针对拥有复杂比较逻辑的对象而言，这是一种性能优化的方式。
  
  2.    使用instanceof操作符检查"参数是否为正确的类型"，如果不是则返回false。
  
  3.    把参数转换成为正确的类型。由于已经通过instanceof的测试，因此不会抛出ClassCastException异常。
  
  4.    对于该类中的每个"关键"域字段，检查参数中的域是否与该对象中对应的域相匹配。
  
  如果以上测试均全部成功返回true，否则false。见如下示例代码：
  
    @Override public boolean equals(Object o) {
        if (o == this) 
            return true;
        
        if (!(o instanceof MyType))
            return false;
            
        MyType myType = (MyType)o;
        return objField.equals(o.objField) && intField == o.intField 
            && Double.compare(doubleField,o.doubleField) == 0 
            && Arrays.equals(arrayField,o.arrayField);
    }
    
  从上面的示例中可以看出，如果域字段为Object对象，则使用equals方法进行两者之间的相等性比较，如果为int等整型基本类型，可以直接比较，如果为浮点型基本类型，考虑到精度和Double.NaN和Float.NaN等问题，推荐使用其对应包装类的compare方法，如果是数组，可以使用JDK 1.5中新增的Arrays.equals方法。众所周知，&&操作符是有短路原则的，因此应该将最有可能不相同和比较开销更低的域比较放在最前面。
  
   最后需要提起注意的是Object.equals的参数类型为Object，如果要重载该方法，必须保持参数列表的一致性，如果我们将子类的equals方法写成:public boolean equals(MyType o)；Java的编译器将会视其为Object.equals的过载(Overload)方法，因此推荐在声明该重载方法时，在方法名的前面加上@Override注释标签，一旦当前声明的方法因为各种原因并没有重载超类中的方法，该标签的存在将会导致编译错误，从而提醒开发者此方法的声明存在语法问题。
   
# 九、覆盖equals时总要覆盖hashCode

 一个通用的约定，如果类覆盖了equals方法，那么hashCode方法也需要被覆盖。否则将会导致该类无法和基于散列的集合一起正常的工作，如HashMap、HashSet。来自JavaSE6的约定如下：
 
      1.    在应用程序执行期间，只要对象的equals方法的比较操作所用到的信息没有被修改，那么对这同一个对象多次调用，hashCode方法都必须始终如一地返回同一个整数。在同一个应用程序的多次执行过程中，每次执行所返回的整数可以不一致。
      2.    如果两个对象根据equals(Object)方法比较是相等的，那么调用这两个对象中任意一个对象的hashCode方法都必须产生同样的整数结果。
      3.    如果两个对象根据equals(Object)方法比较是不相等的，那么调用这两个对象中任意一个对象的hashCode方法，则不一定要产生不同的整数结果。但是程序员应该知道，给不相等的对象产生截然不同的整数结果，有可能提高散列表的性能。
      
   如果类没有覆盖hashCode方法，那么Object中缺省的hashCode实现是基于对象地址的，就像equals在Object中的缺省实现一样。如果我们覆盖了equals方法，那么对象之间的相等性比较将会产生新的逻辑，而此逻辑也应该同样适用于hashCode中散列码的计算，既参与equals比较的域字段也同样要参与hashCode散列码的计算。见下面的示例代码：
   
     public final class PhoneNumber {
        private final short areaCode;
        private final short prefix;
        private final short lineNumber;
        public PhoneNumber(int areaCode,int prefix,int lineNumber) {
            //做一些基于参数范围的检验。
            this.areaCode = areaCode;
            this.prefix = prefix;
            this.lineNumber = lineNumber;
        }
        @Override public boolean equals(Object o) {
            if (o == this)
                return true;
            if (!(o instanceof PhoneNumber)) 
                return false;
            PhoneNumber pn = (PhoneNumber)o;
            return pn.lineNumber = lineNumber && pn.prefix == prefix && pn.areaCode = areaCode;
        }
    }
    public static void main(String[] args) {
        Map<PhoneNumber,String> m = new HashMap<PhoneNumber,String>();
        PhoneNumber pn1 = new PhoneNumber(707,867,5309);
        m.put(pn1,"Jenny");
        PhoneNumber pn2 = new PhoneNumber(707,867,5309);
        if (m.get(pn) == null)
            System.out.println("Object can't be found in the Map");
    }
    
从以上示例的输出结果可以看出，新new出来的pn2对象并没有在Map中找到，尽管pn2和pn1的相等性比较将返回true。这样的结果很显然是有悖我们的初衷的。如果想从Map中基于pn2找到pn1，那么我们就需要在PhoneNumber类中覆盖缺省的hashCode方法，见如下代码：

    @Override public int hashCode() {
        int result = 17;
        result = 31 * result + areaCode;
        result = 31 * result + prefix;
        result = 31 * result + lineNumber;
        return result;
    }
    
在上面的代码中，可以看到参与hashCode计算的域字段也同样参与了PhoneNumber的相等性(equals)比较。对于生成的散列码，推荐不同的对象能够尽可能生成不同的散列，这样可以保证在存入HashMap或HashSet中时，这些对象被分散到不同的散列桶中，从而提高容器的存取效率。对于有些不可变对象，如果需要被频繁的存取于哈希集合，为了提高效率，可以在对象构造的时候就已经计算出其hashCode值，hashCode()方法直接返回该值即可，如：

    public final class PhoneNumber {
        private final short areaCode;
        private final short prefix;
        private final short lineNumber;
        private final int myHashCode;
        public PhoneNumber(int areaCode,int prefix,int lineNumber) {
            //做一些基于参数范围的检验。
            this.areaCode = areaCode;
            this.prefix = prefix;
            this.lineNumber = lineNumber;
            myHashCode = 17;
            myHashCode = 31 * myHashCode + areaCode;
            myHashCode = 31 * myHashCode + prefix;
            myHashCode = 31 * myHashCode + lineNumber;
        }
        @Override public boolean equals(Object o) {
            if (o == this)
                return true;
            if (!(o instanceof PhoneNumber)) 
                return false;
            PhoneNumber pn = (PhoneNumber)o;
            return pn.lineNumber = lineNumber && pn.prefix == prefix && pn.areaCode = areaCode;
        }
        @Override public int hashCode() {
            return myHashCode;
        }
    }
    
另外，该条目还建议不要仅仅利用某一域字段的部分信息来计算hashCode，如早期版本的String，为了提高计算哈希值的效率，只是挑选其中16个字符参与hashCode的计算，这样将会导致大量的String对象具有重复的hashCode，从而极大的降低了哈希集合的存取效率。
      
      
# 十、始终要覆盖toString

与equals和hashCode不同的是，该条目推荐应该始终覆盖该方法，以便在输出时可以得到更明确、更有意义的文字信息和表达格式。这样在我们输出调试信息和日志信息时，能够更快速的定位出现的异常或错误。如上一个条目中PhoneNumber的例子，如果不覆盖该方法，就会输出PhoneNumber@163b91 这样的不可读信息，因此也不会给我们诊断问题带来更多的帮助。以下代码重载了该方法，那么在我们调用toString或者println时，将会得到"(408)867-5309"。

    @Override String toString() {
        return String.format("(%03d) %03d-%04d",areaCode,prefix,lineNumber);
     }
     
 对于toString返回字符串中包含的域字段，如本例中的areaCode、prefix和lineNumber，应该在该类(PhoneNumber)的声明中提供这些字段的getter方法，以避免toString的使用者为了获取其中的信息而不得不手工解析该字符串。这样不仅带来不必要的效率损失，而且在今后修改toString的格式时，也会给使用者的代码带来负面影响。提到toString返回字符串的格式，有两个建议，其一是尽量不要固定格式，这样会给今后添加新的字段信息带来一定的束缚，因为必须要考虑到格式的兼容性问题，再者就是推荐可以利用toString返回的字符串作为该类的构造函数参数来实例化该类的对象，如BigDecimal和BigInteger等装箱类。
 
   这里还有一点建议是和hashCode、equals相关的，如果类的实现者已经覆盖了toString的方法，那么完全可以利用toString返回的字符串来生成hashCode，以及作为equals比较对象相等性的基础。这样的好处是可以充分的保证toString、hashCode和equals的一致性，也降低了在对类进行修订时造成的一些潜在问题。尽管这不是刚性要求的，却也不失为一个好的实现方式。该建议并不是源于该条目，而是去年在看effective C#中了解到的。
   
# 十一 谨慎地覆盖clone

Clone提供一种语言之外的机制：无需调用构造器就可以创建对象。

它的通用约定非常弱：

创建和返回该对象的一个拷贝。这个拷贝的精确含义取决于该对象的类。一般含义是，对于任何对象x，表达式x.clone() != x 将会是true，并且，表达式x.clone().getClass() == x.getClass() 将会是true，但这些不是绝对的要求，通常情况下，表达式 x.clone().equals(x) 将会是true，这也不是一个绝对的要求，拷贝对象往往是创建它的类的一个新实例，但它同时也会要求拷贝内部的数据结构。

如果类的每个域包含一个基本类型的值，或者包含一个指向不可变对象的引用，那么被返回的对象则正是所需要的对象，如PhoneNumber类：

    public class PhoneNumber implements Cloneable{
    private final int areaCode;
    private final int prefix;
    private final int lineNumber;
    
    public PhoneNumber(int areaCode, int prefix, int lineNumber) {
        rangeCheck(areaCode, 999, "area code");
        rangeCheck(prefix, 999, "prefix");
        rangeCheck(lineNumber, 9999, "line number");
        this.areaCode = areaCode;
        this.prefix = prefix;
        this.lineNumber = lineNumber;
    }
    
    private static void rangeCheck(int arg, int max, String name) {
        if(arg < 0 || arg > max) {
            throw new IllegalArgumentException(name + ": " + arg);

        }
    }
    
    @Override
    public boolean equals(Object o) {
        if(o == this)
            return true;
        if(!(o instanceof PhoneNumber))
            return false;
        PhoneNumber pn = (PhoneNumber)o;
        return pn.lineNumber == lineNumber
                && pn.prefix == prefix
                && pn.areaCode == areaCode;
    }
    
    @Override
    public PhoneNumber clone() {
        try {
            return (PhoneNumber) super.clone();
        } catch(CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
    }
   
只需要简单地调用super.clone() 而不用做进一步的处理。

如果对象中包含的域引用了可变的对象：

    public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITAL_CAPACITY = 16;
    
    public Stack() {
        elements = new Object[DEFAULT_INITAL_CAPACITY];
    }
    
    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }
    
    public Object pop() {
        if(size == 0) {
            throw new EmptyStackException();
        }
        Object result = elements[--size];
        elements[size] = null;
        return result;
    }
    
    private void ensureCapacity() {
        if(elements.length == size)
            elements = Arrays.copyOf(elements, 2 * size + 1);
    }
    }
    
如果把这个类做成是可克隆的。如果它的clone方法仅仅返回super.clone()，这样得到的Stack实例，在size域有正确的值，但它的elements域将引用与原始Stack实例相同的数组。

为了使Stack类中的clone方法正常地工作，必须拷贝栈的内部信息：

    @Override
    public Stack clone() {
    try {
        Stack result = (Stack) super.clone();
        result.elements = elements.clone();
        return result;
    } catch (CloneNotSupportedException e) {
        throw new AssertionError();
     }
    }
    
如果elements域是final的，上面的clone方法无法正常工作，因为clone方法被禁止给elements赋新值。

因此，实现一个clone操作需要以下几个步骤：

1 实现Cloneable接口；

2 override clone方法；

3 在clone方法中递归调用引用类型字段的clone方法（需要这些类型实现步骤1和2),并给目标对象赋值，返回目标对象。

另一个实现对象拷贝的好办法是提供一个拷贝构造器或拷贝工厂。拷贝构造器只是一个构造器，它唯一的参数类型是包含该构造器的类，例如 public Yum(Yum yun);

拷贝工厂是类似于拷贝构造器的静态工厂：public static Yum newInstance(Yum yum);

它们不依赖于有风险的、语言之外的对象创建机制，也不要求遵守尚未制定好文档的规范，不会与final域的正常使用发生冲突，不抛出不必要的受检异常，不需要进行类型转换。

拷贝构造器或者拷贝工厂可以带一个参数，参数类型是通过该类实现的接口，例如所有通用集合实现都提供一个拷贝构造器，它的参数类型是Collection或者Map。基于接口的拷贝构造器和拷贝工厂允许客户选择拷贝的实现类型，例如假设有一个HashSet s，希望把它拷贝成一个TreeSet，clone方法无法提供这样的功能，但用转换构造器实现：new TreeSet(s)。 

cloneable的问题导致我们不应该扩展这个接口，为了继承而设计的类也不应该实现这个接口，由于它具有这么多缺点，专家级的程序员从来不去覆盖clone方法， 也从来不去调用它，除非拷贝数组。

对于一个专门为继承而设计的类，如果未能提供行为良好的受保护clone方法，它的子类就不可能实现Cloneable接口。

# 十二、考虑实现Comparable接口

 和之前提到的通用方法equals、hashCode和toString不同的是compareTo方法属于Comparable接口，该接口为其实现类提供了排序比较的规则，实现类仅需基于内部的逻辑，为compareTo返回不同的值，既A.compareTo(B) > 0可视为A > B，反之则A < B，如果A.compareTo(B) == 0，可视为A == B。在C++中由于提供了操作符重载的功能，因此可以直接通过重载操作符的方式进行对象间的比较，事实上C++的标准库中提供的缺省规则即为此，如bool operator>(OneObject o)。在Java中，如果对象实现了Comparable接口，即可充分利用JDK集合框架中提供的各种泛型算法，如：Arrays.sort(a); 即可完成a对象数组的排序。事实上，JDK中的所有值类均实现了该接口，如Integer、String等。
 
   Object.equals方法的通用实现准则也同样适用于Comparable.compareTo方法，如对称性、传递性和一致性等，这里就不做过多的赘述了。然而两个方法之间有一点重要的差异还是需要在这里提及的，既equals方法不应该抛出异常，而compareTo方法则不同，由于在该方法中不推荐跨类比较，如果当前类和参数对象的类型不同，可以抛出ClassCastException异常。在JDK 1.5 之后我们实现的Comparable<T>接口多为该泛型接口，不在推荐直接继承1.5 之前的非泛型接口Comparable了，新的compareTo方法的参数也由Object替换为接口的类型参数，因此在正常调用的情况下，如果参数类型不正确，将会直接导致编译错误，这样有助于开发者在coding期间修正这种由类型不匹配而引发的异常。
   
   在该条目中针对compareTo的相等性比较给出了一个强烈的建议，而不是真正的规则。推荐compareTo方法施加的等同性测试，在通常情况下应该返回和equals方法同样的结果，考虑如下情况：
   
   
     public static void main(String[] args) {
        HashSet<BigDecimal> hs = new HashSet<BigDecimal>();
        BigDecimal bd1 = new BigDecimal("1.0");
        BigDecimal bd2 = new BigDecimal("1.00");
        hs.add(bd1);
        hs.add(bd2);
        System.out.println("The count of the HashSet is " + hs.size());
        
        TreeSet<BigDecimal> ts = new TreeSet<BigDecimal>();
        ts.add(bd1);
        ts.add(bd2);
        System.out.println("The count of the TreeSet is " + ts.size());
    }
    /*    输出结果如下：
        The count of the HashSet is 2
        The count of the TreeSet is 1
    */
    
由以上代码的输出结果可以看出，TreeSet和HashSet中包含元素的数量是不同的，这其中的主要原因是TreeSet是基于BigDecimal的compareTo方法是否返回0来判断对象的相等性，而在该例中compareTo方法将这两个对象视为相同的对象，因此第二个对象并未实际添加到TreeSet中。和TreeSet不同的是HashSet是通过equals方法来判断对象的相同性，而恰恰巧合的是BigDecimal的equals方法并不将这个两个对象视为相同的对象，这也是为什么第二个对象可以正常添加到HashSet的原因。这样的差异确实给我们的编程带来了一定的负面影响，由于HashSet和TreeSet均实现了Set<E>接口，倘若我们的集合是以Set<E>的参数形式传递到当前添加BigDecimal的函数中，函数的实现者并不清楚参数Set的具体实现类，在这种情况下不同的实现类将会导致不同的结果发生，这种现象极大的破坏了面向对象中的"里氏替换原则"。

   在重载compareTo方法时，应该将最重要的域字段比较方法比较的最前端，如果重要性相同，则将比较效率更高的域字段放在前面，以提高效率，如以下代码：
   
     public int compareTo(PhoneNumer pn) {
        if (areaCode < pn.areaCode)
            return -1;
        if (areaCode > pn.areaCode)
            return 1;
            
        if (prefix < pn.prefix)
            return -1;
        if (prefix > pn.prefix)
            return 1;
            
        if (lineNumber < pn.lineNumer)
            return -1;
        if (lineNumber > pn.lineNumber)
            return 1;
        return 0;
    }
    
 上例给出了一个标准的compareTo方法实现方式，由于使用compareTo方法排序的对象并不关心返回的具体值，只是判断其值是否大于0，小于0或是等于0，因此以上方法可做进一步优化，然而需要注意的是，下面的优化方式会导致数值类型的作用域溢出问题。
 
     public int compareTo(PhoneNumer pn) {
        int areaCodeDiff = areaCode - pn.areaCode;
        if (areaCodeDiff != 0)
            return areaCodeDiff;
        int prefixDiff = prefix - pn.prefix;
        if (prefixDiff != 0)
            return prefixDiff;
    
        int lineNumberDiff = lineNumber - pn.lineNumber;
        if (lineNumberDiff != 0)
            return lineNumberDiff;
        return 0;
    }