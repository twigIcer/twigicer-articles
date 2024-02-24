# Java字符串遍历时，toCharArray()方法与charAt()方法效率比较

## 问题提出：

最近在力扣刷题时，发现在字符串遍历时，大佬们通常会使用 toCharArray () 方法，而很少使用 harAt () 方法，于是我就挺好奇这两种方法到底谁的效率会高一点呢？我就去跑了一些测试案例，但是结果却是时不时 toCharArray () 效率高，时不时是 charAt () 效率高，我觉得可能是我的测试案例的字符串长度不够，但是我字符串长度加到我认为足够长时，我发现 charAt () 的效率明显高于 toCharArray ()。

```java
public class Test {
    public static void main(String[] args) {
        String s = "这里面的字符串内容就省略了，知道它足够长就可以";
 
        long st1 = System.currentTimeMillis();
        char[] ch = s.toCharArray();
        for (char c : ch) {
            System.out.print(c);
        }
        long et1 = System.currentTimeMillis();
        long rt1 = et1 - st1;
        System.out.println();
        System.out.println(rt1);
 
        System.out.println("-------------------------------------------------");
 
        long st2 = System.currentTimeMillis();
        for (int i = 0; i < s.length(); i++) {
            System.out.print(s.charAt(i));
        }
        long et2 = System.currentTimeMillis();
        long rt2 = et2 - st2;
        System.out.println();
        System.out.println(rt2);
    }
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)![img](https://img-blog.csdnimg.cn/df5adb0187f54eaa8d42d45aab7cb722.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

注意：我有多次测试，结果就是 toCharAray () 方法 30ms 左右，charAt () 方法 15ms 左右。

我就挺好奇，既然 charAt () 方法效率高，那为什么用 toCharArray () 方法的人多呢？难道是 charAt () 方法有什么弊端吗？

## 探究过程：

于是我就询问了黑马的满老师，而满老师给我的回答是：” 我觉得没啥弊端吧，java api 层面的效率问题在我看来不是太重要 “。其实这个回答并不能完全解答我的疑惑，既然没有什么弊端，效率还高，那么在力扣着这种看重效率的平台，为什么很少使用 charAt () 方法呢？

然后我去网上搜了这个问题，但是回答能各执一词，没能很好的说服我，但是我发现很多人认为是 toCharArray () 方法效率高，但是这和我测试结果相悖啊。

然后我去 CSDN 发布了问答，可惜等来的只有一个回答，那就是：明显 toCharArray () 方法效率高，因为直接访问数组，但是 toCharArray () 本身是需要时间和额外内存的。这个回答怎么说呢，不太能让我信服。

碰巧在我复习 String 相关知识时，看见黑马的视频中说：使用 toCharArray () 方法效率会高，因为在使用 charAt () 方法遍历时会调用其他一些方法，这些方法入栈出栈会花时间。确实这个说法有道理，可是为什么和我测试结果不一致呢？

## 应该是正解：

最后我想到了 chatGPT，事实也证明，chatGPT 确实给力，至少它的回答让我信服了，它的回答如下：

> ” 在 Java 中，字符串的 `charAt()` 方法和 `toCharArray()` 方法都可以用于遍历字符串，但它们的效率取决于具体的使用情况。
>
> 如果只需要遍历字符串中的单个字符，使用 `charAt()` 方法更为简便，因为它只需要访问字符串中的一个字符，因此在这种情况下，`charAt()` 方法的效率更高。
>
> 然而，如果需要遍历整个字符串并进行一些操作，例如将字符串转换为字符数组以进行某些处理，则使用 `toCharArray()` 方法可能更为高效。这是因为 `toCharArray()` 方法可以一次将整个字符串转换为字符数组，使得我们可以在数组上进行一系列的操作，比如修改、排序等等。而使用 `charAt()` 方法则需要反复访问字符串，效率较低。
>
> 综合来说，如果只需要遍历单个字符，使用 `charAt()` 方法更为高效，而如果需要对整个字符串进行操作，则使用 `toCharArray()` 方法可能更为高效。但在实际应用中，对于小型字符串，两种方法的差异并不明显，因此选择哪种方法取决于具体的情况和个人偏好。“

不得不说它的回答很中肯，也符合我的测试案例，至于力扣用 `toCharArray()` 方法，也确实是他们并不是简单的遍历字符串，还有其他的操作。所以在我心里这个回答应该是正解。

## 总结：

关于 Java 字符串遍历时，toCharArray () 方法与 charAt () 方法效率比较，我比较支持的观点为：

如果只需要遍历单个字符，使用 `charAt()` 方法更为高效，而如果需要对整个字符串进行操作，则使用 `toCharArray()` 方法可能更为高效。

但是对于小型字符串，两种方法的差异并不明显，也许正如满老师所说的：java api 层面的效率问题不是太重要吧。