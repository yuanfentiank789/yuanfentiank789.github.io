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
        
 对于上例，如果执行cis.equals(s)将会返回true，因为在该class的equals方法中对参数o的类型针对String作了特殊的判断和特殊的处理，因此如果equals中传入的参数类型为String时，可以进一步完成大小写不敏感的比较。然而在String的equals中，并没有针对CaseInsensitiveString类型做任何处理，因此s.equals(cis)将一定返回false。针对该示例代码，由于无法确定List.contains的实现是基于cis.equals(s)还是基于s.equals(cis)，对于实现逻辑两者都是可以接受的，既然如此，外部的使用者在调用该方法时也应该同样保证并不依赖于底层的具体实现逻辑。由此可见，equals方法的对称性是非常必要的。以上的equals实现可以做如下修改：

