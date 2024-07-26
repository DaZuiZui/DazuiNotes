# Java字符串怎么找到里面的中文

可以使用正则表达式来查找字符串中的中文字符。因为中文字符的Unicode范围是u4e00,到u9fa5。

如果这时候我们变了需求要找韩文，那就是uAC00到ud7A3，不管什么语言都有一个范围的。

~~~java
import java.util.regex.Matcher;
import java.util.regex.Pattern;

public class ChineseCharacterFinder {
    public static void main(String[] args) {
        String input = "Hello 你好 World 世界";

        // 定义匹配中文字符的正则表达式
        String regex = "[\\u4e00-\\u9fa5]";
        Pattern pattern = Pattern.compile(regex);
        Matcher matcher = pattern.matcher(input);

        // 查找并打印所有的中文字符
        while (matcher.find()) {
            System.out.println("Found Chinese character: " + matcher.group());
        }
    }
}
~~~

