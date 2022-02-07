# number-range-expression

这里是一段 Java 编写的表达式解析代码，我们先通过实例来演示该表达式的效果。当然实际上这并不是性能最高的解析方式。

```java
import java.util.Arrays;

public class Main {
    public static void main(String[] args) {
        System.out.println(Arrays.toString(parse("1644217086053 -10 *1 5|0 -100 1000 -*1 *2", new Long[]{10L, 20L})));
    }

    private static Long[] parse(String expression, Long[] wedges) {
        var split = expression.split("\\|");
        if (split.length != 2) {
            throw new IllegalArgumentException(expression);
        }
        var start = split[0].split(" ");
        var end = split[1].split(" ");
        var startPoint = Long.parseLong(start[0]);
        for (int i = 1; i < start.length; i++) {
            startPoint = startPoint + parseOffset(start[i], wedges);
        }
        var endPoint = Long.parseLong(end[0]);
        if (endPoint == 0) {
            endPoint = startPoint;
        }
        for (int i = 1; i < end.length; i++) {
            endPoint = endPoint + parseOffset(end[i], wedges);
        }
        return new Long[]{startPoint, endPoint};
    }

    private static Long parseOffset(String offsetExpression, Long[] wedges) {
        var negative = false;
        if (offsetExpression.startsWith("-")) {
            negative = true;
            offsetExpression = offsetExpression.substring(1);
        }
        var offset = 0L;
        if (offsetExpression.startsWith("*")) {
            offset = wedges[Integer.parseInt(offsetExpression.substring(1)) - 1];
        } else {
            offset = Long.parseLong(offsetExpression);
        }
        return negative ? -offset : offset;
    }
}
```

这段代码最后会输出：

```
[1644217086058, 1644217086968]
```



我们来解释这个表达式的含义，参考代码来理解就更加容易理解了。这个表达式中间有一个 | 符号用于分割表达式，左侧表达式我们称之为起始值表达式，右侧表达式我们称之为截止值表达式，这两个表达式都是由数字、符号（-/*）和空格组成，这两个表达式再使用空格分割又会得到多个表达式，这些表达式最终会解析成一个数字，我们分别累加这两组数字就可以得到两个数字，它们可以表达一个数字区间。

用空格分割起始值表达式，会得到多个表达式，其中从左开始的第一个表达式应该是一个数字（例子中是 1644217086053），- 开始的表达式表示最后我们会取反这个数字（表达负数），包含 * 的表达式表示这是一个占位符，我们会从外部获得一个数字替换它（*后面的数字是替换的索引）。累加解析完的数字得到了起始值。

同样用空格分割截止值表达式，也得到了多个表达式，从左开始的第一个表达式也应该是一个数字，如果为 0 的情况下，我们将会使用计算出来的起始值替换它，其它的情况与起始值表达式规则一样。累加解析完的数字得到了截止值。

根据上文的规则尝试解析这个表达式。起始值表达式，1644217086053+（-10）+（*1）+5， *1 被替换为 10，最终起始值是 1644217086058。截止值表达式，0+ （-100）+（1000）+（- *1）+（ *2）, 0 被替换为 1644217086058，- *1 被替换为 -10， *2 被替换为 20，最终截止值是 1644217086968。
