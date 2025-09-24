**阅读目录(Content)**

* [1.中序(中缀)&前序表达式](#_label0)
* [2.后序表达式(逆波兰式)](#_label1)

+ [2.1 中序转后序表达式](#_lab2_1_0)
+ [2.2 求解后序表达式](#_lab2_1_1)
+ [2.3 双栈法直接求值](#_lab2_1_2):[westworld加速](https://xbsj9.com)

在游戏开发中，我们经常需要在配置表中定义各种公式，比如 `a * (b + c)`，用来计算技能伤害、属性加成等。如果直接让程序在运行时解析并执行这些公式，就需要处理运算符优先级和括号等复杂问题。

这时，后序表达式就派上了用场。我们将中序表达式 `a * (b + c)` 转换为后序表达式 `a b c + *`，这样程序只需要一个栈就能高效计算，无需担心优先级和括号。

而要深入理解后序表达式，就不得不提到与之相关的完整概念体系：前序表达式、中序表达式和后序表达式。这三种表达式各有特点，共同构成了计算机处理数学表达式的理论基础。

# 1.中序(中缀)&前序表达式

中序表达式就是日常书写的公式，但这种日常公式对计算机并不友好，有括号、优先级等需要考虑，计算机直接读取较为麻烦

因此波兰数学家扬·武卡谢维奇发明了"波兰表示法"（前序表达式）

所以前序表达式又叫做 波兰式

例如LISP的语法风格就是基于前序表达式的：

```
;; 中序: 1 + 2 * 3
;; 前序: 
+ 1 * 2 3    ; => 7

;; 中序: (1 + 2) * 3  
;; 前序:
* + 1 2 3   ; => 9
```

而我们现在使用较多的则是后序表达式，因为中序表达式转换为后序表达式内存开销较少，计算更为友好。

# 2.后序表达式(逆波兰式)

几个逆波兰式的例子：

```
中序：1 + 2
后序：1 2 +

中序：1 + 2 * 3
后序：1 2 3 * +

中序：(1 + 2) * 3
后序：1 2 + 3 *
```

通常可以将表达式从中序转后序后存放在内存里，供计算时使用。也可以通过双栈法直接计算出结果。

常见的中序转后序算法如下。

## 2.1 中序转后序表达式

示例公式：(A + B) \* C

首先创建栈结构存放符号

从左向右读取原公式，读到非符号输出，读到符号以如下规则操作：

* 若栈为空，读到符号可以直接入栈
* \* / 优先级高于 + -，将读取到的符号与栈顶符号做比较，若<=栈顶符号，则出栈(循环操作，直到条件不满足)，并输出
* 若读取到左括号(，则连同括号一起入栈。若读取到右括号)，则循环出栈，直到找到左括号位置
* 若所有内容读取完，则输出栈的内容

注：输出是指将值添加至返回值字符串或其他数据格式

参照上面写的示例公式，操作流程如下：

1. 读 `(`  入栈
   栈：`(`
   输出：空
2. 读 `A`  输出
   输出：`A`
3. 读 `+`  入栈
   栈：`(, +`
   输出：`A`
4. 读 `B` 输出
   输出：`A B`
5. 读 `)` 弹出直到遇到 `(` → 弹出 `+`
   栈：空
   输出：`A B +`
6. 读 `*` 栈空 ， 入栈
   栈：`*`
   输出：`A B +`
7. 读 `C` 输出
   输出：`A B + C`

结束后弹栈：输出 `A B + C *`

通常游戏配表中的公式可运用到此方法，以带变量公式为例，进行代码演示

1 + 2 \* 3 + (4 \* e + f) \* g

代码如下：

```
ConvertRpn("1 + 2 * 3 + (4 * e + f) * g", out var rpnStr);
Debug.Log(rpnStr);//1 2 3 * + 4 e * f + g * +
```

```
void ConvertRpn(string inStr, out string rpnStr)
{
    int GetPriority(char ch)
    {
        return ch switch
        {
            '+' or '-' => 1,
            '*' or '/' => 2,
            '(' => 0,    // 左括号在栈内优先级最低
            _ => -1      // 其他情况（包括右括号）
        };
    }

    var rpnBuilder = new StringBuilder();
    var stack = new Stack<char>();

    var numBuilder = new StringBuilder();

    foreach (var ch in inStr)
    {
        if (Char.IsWhiteSpace(ch))
        {
            // 遇到空格时，如果正在构建数字，就完成这个数字
            if (numBuilder.Length > 0)
            {
                rpnBuilder.Append(numBuilder.ToString()).Append(' ');
                numBuilder.Clear();
            }
            continue;
        }

        if (char.IsDigit(ch) || ch == '.')
        {
            // 数字或小数点：添加到数字构建器
            numBuilder.Append(ch);
            continue;
        }
        else
        {
            // 遇到操作符前，先完成当前数字的构建
            if (numBuilder.Length > 0)
            {
                rpnBuilder.Append(numBuilder.ToString()).Append(' ');
                numBuilder.Clear();
            }
        }

        switch (ch)
        {
            case '(':
                stack.Push(ch);
                break;

            case ')':
                // 弹出直到遇到左括号
                while (stack.Count > 0 && stack.Peek() != '(')
                {
                    rpnBuilder.Append(stack.Pop()).Append(' ');
                }
                if (stack.Count > 0 && stack.Peek() == '(')
                {
                    stack.Pop(); // 弹出左括号但不输出
                }
                break;

            case '+':
            case '-':
            case '*':
            case '/':
                // 处理操作符优先级
                while (stack.Count > 0 && stack.Peek() != '(' &&
                        GetPriority(ch) <= GetPriority(stack.Peek()))
                {
                    rpnBuilder.Append(stack.Pop()).Append(' ');
                }
                stack.Push(ch);
                break;

            default:
                rpnBuilder.Append(ch).Append(' ');
                break;
        }
    }

    // 处理最后一个数字（如果存在）
    if (numBuilder.Length > 0)
    {
        rpnBuilder.Append(numBuilder.ToString()).Append(' ');
    }

    // 弹出栈中剩余操作符
    while (stack.Count > 0)
    {
        rpnBuilder.Append(stack.Pop()).Append(' ');
    }

    // 移除末尾多余的空格
    if (rpnBuilder.Length > 0 && rpnBuilder[rpnBuilder.Length - 1] == ' ')
    {
        rpnBuilder.Length--;
    }

    rpnStr = rpnBuilder.ToString();
}
```

 这样就可以将公式转换为后序表达式先存放在内存中了。

## 2.2 求解后序表达式

求解后序表达式，同样需要一个栈结构来辅助。

规则是：

1. **遇到操作数（数字/变量）** → 压入栈。
2. **遇到运算符** → 从栈中弹出两个操作数：

   * **先弹出的是右操作数**
   * **再弹出的是左操作数**
     然后进行运算，把结果压回栈。
3. 扫描完后，栈顶就是最终结果

代码如下：

```
rpnStr = "1 2 3 * + 4 e * f + g * +";
var result = CalcPrn(rpnStr, new Dictionary<char, float> { ['e'] = 10f, ['f'] = 12f, ['g'] = 14f });
Debug.Log(result);// 735
```

```
float CalcPrn(string rpnStr, Dictionary<char, float> replace)
{
    var rpnBuilder = new StringBuilder();
    var stack = new Stack<object>();

    var numBuilder = new StringBuilder();

    foreach (var ch in rpnStr)
    {
        if (Char.IsWhiteSpace(ch))
        {
            if (numBuilder.Length > 0)
            {
                stack.Push(float.Parse(numBuilder.ToString()));
                numBuilder.Clear();
            }
            continue;
        }

        if (char.IsDigit(ch) || ch == '.')
        {
            numBuilder.Append(ch);
            continue;
        }
        else
        {
            if (numBuilder.Length > 0)
            {
                stack.Push(float.Parse(numBuilder.ToString()));
                numBuilder.Clear();
            }
        }

        if (ch is '+' or '-' or '*' or '/')
        {
            var x = (float)stack.Pop();
            var y = (float)stack.Pop();

            switch (ch)
            {
                case '+':
                    stack.Push(x + y);
                    break;
                case '-':
                    stack.Push(x - y);
                    break;
                case '*':
                    stack.Push(x * y);
                    break;
                case '/':
                    stack.Push(x / y);
                    break;
            }
        }
        else // 变量
        {
            stack.Push(replace[ch]);
        }
    }

    return (float)stack.Pop();
}
```

## 2.3 双栈法直接求值

使用双栈法，可以不用转换为后序表达式再求值，而是直接求值。

核心思想是使用2个栈，数值栈和字符栈。当字符栈弹出时，顺便就计算当前步骤结果并且放回数值栈。

代码如下：

```
var result = CalcPrnDoubleStack("1 + 2 * 3 + (4 * e + f) * g", new Dictionary<char, float> { ['e'] = 10f, ['f'] = 12f, ['g'] = 14f });
Debug.Log(result);// 735
```

```
 float CalcPrnDoubleStack(string inStr, Dictionary<char, float> replace)
 {
     int GetPriority(char ch)
     {
         return ch switch
         {
             '+' or '-' => 1,
             '*' or '/' => 2,
             '(' => 0,    // 左括号在栈内优先级最低
             _ => -1      // 其他情况（包括右括号）
         };
     }

     float Calc(float x, float y, char token)
     {
         return token switch
         {
             '+' => x + y,
             '-' => x - y,
             '*' => x * y,
             '/' => x / y,
             _ => default
         };
     }
     var stackNum = new Stack<float>();
     var stackToken = new Stack<char>();

     var numBuilder = new StringBuilder();

     foreach (var ch in inStr)
     {
         if (Char.IsWhiteSpace(ch))
         {
             // 遇到空格时，如果正在构建数字，就完成这个数字
             if (numBuilder.Length > 0)
             {
                 stackNum.Push(float.Parse(numBuilder.ToString()));
                 numBuilder.Clear();
             }
             continue;
         }

         if (char.IsDigit(ch) || ch == '.')
         {
             // 数字或小数点：添加到数字构建器
             numBuilder.Append(ch);
             continue;
         }
         else
         {
             // 遇到操作符前，先完成当前数字的构建
             if (numBuilder.Length > 0)
             {
                 stackNum.Push(float.Parse(numBuilder.ToString()));
                 numBuilder.Clear();
             }
         }

         switch (ch)
         {
             case '(':
                 stackToken.Push(ch);
                 break;

             case ')':
                 while (stackToken.Count > 0 && stackToken.Peek() != '(')
                 {
                     var token = stackToken.Pop();
                     var y = stackNum.Pop();
                     var x = stackNum.Pop();
                     stackNum.Push(Calc(x, y, token));
                 }
                 stackToken.Pop();
                 break;

             case '+':
             case '-':
             case '*':
             case '/':
                 while (stackToken.Count > 0 && stackToken.Peek() != '(' &&
                     GetPriority(ch) <= GetPriority(stackToken.Peek()))
                 {
                     var token = stackToken.Pop();
                     var y = stackNum.Pop();
                     var x = stackNum.Pop();
                     stackNum.Push(Calc(x, y, token));
                 }
                 stackToken.Push(ch);
                 break;

             default:

                 stackNum.Push(replace[ch]);

                 break;
         }
     }

     while (stackToken.Count > 0)
     {
         var token = stackToken.Pop();
         var y = stackNum.Pop();
         var x = stackNum.Pop();
         stackNum.Push(Calc(x, y, token));
     }

     return stackNum.Pop();
 }
```
