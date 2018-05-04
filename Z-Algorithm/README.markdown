# Z-Algorithm 字符串搜索

目标：给定一个模式串，用 Swift 写一个线性时间复杂度的匹配算法，返回在字符串中出现的位置。

换而言之，我们需要实现一个 `String` 的扩展方法 `indexsOf(pattern:String)`, 能够返回一个 `[Int]` 数组代表所有模式串出现的索引位置，或者返回 `nil` 如果没有在字符串中找到。

举个例子:

```swift
let str = "Hello, playground!"
str.indexesOf(pattern: "ground")   // Output: [11]

let traffic = "🚗🚙🚌🚕🚑🚐🚗🚒🚚🚎🚛🚐🏎🚜🚗🏍🚒🚲🚕🚓🚌🚑"
traffic.indexesOf(pattern: "🚑") // Output: [4, 21]
```

许多字符串搜索算法都会有一个预处理函数计算一个表用在随后的计算过程中。这个表可以可以节省模式串匹配阶段的一些时间，因为可以避免一些不必要的字符串比较。Z-Algorithm 就是众多预处理函数的一种。尽管它生为预处理函数（在 [KMP](../Knuth-Morris-Pratt/) 算法和其他算法中就承担了一个这样的角色），但是本文将介绍如何将它作为字符串搜索算法使用。

### Z-Algorithm 模式串的前缀

正如本文所说，Z-Algorithm 是算法开始部分通过处理模式串计算出一个跳过非必要比较的表。Z-Algorithm 计算模式串后得到一个整数数组（文献中称之为 `Z`）每个元素称作 `Z[i]`, 表示模式串 `P` 的以 `i` 开始的最长子字符串的前缀与 `P` 的前缀相匹配的长度。简而言之就是 `Z[i]` 记录了 `P[i...|P|]` 最长的与 `P` 前缀相同的前缀。举个例子，假设 `P = "ffgtrhghhffgtggfredg"`。那么 `z[5] =0 (f...h...)`，`z[9] = 4 (ffgtr...ffgtg...)` 和 `z[15] = 1 (ff..fr..)`。（译者注：好吧，这个例子其实很难看，相信你可能数的眼都花了，这里 `z[5] = hghhffgtggfredg` 与原字符串比较前缀一个都没有所以结果为0，而 `z[9] = ffgtggfredg` 与原字符串比较一下结果为 `ffgt` 相同，结果为 4。）

但是我们如何计算 `Z `? 在介绍这个算法之前，我们需要先介绍一个下 Z-box 这个概念。 一个 Z-Box  含有 `(left,right)` 一对值，用来在计算过程中记录子字符串与 `P` 前缀相同的长度。`left` 和 `right` 这两个索引值各自代表子字符串的左边界和右边界索引。Z-Algorithm 定义比较感性，它从 `k-1` 开始，计算了模式串中每个位置 `k`。算法背后的思想是之前计算的值可以加快 `Z[k + 1]` 的演算，避免重复已经比较过的。思考一下：如果迭代到 `k = 100`, 分析模式串 `100` 位置如何计算。所有的 `Z[1]` 到 `Z[99]` 已经计算过并且 `left = 70`, `right = 120`。这意味着子字符串长度为 `51` 且是从 `70` 开始到 `120`结束，而且还是与模式串前缀相匹配的。推理一下后可以认为从 `100` 开始，长度为 `21` 的字符与模式串中从 `30` 开始长度为 `21` 的子字符串相匹配（因为我们是在一个与模式串前缀相匹配的子字符串中）。因此我们可以避免额外的比较直接用 `Z[30]` 来计算 `Z[100]`。

这是这个算法背后的简单思想。无法通过之前计算的值直接进行处理的情况很少，但还是有一些比较需要处理一下。

下面是计算 `Z-array` 的代码：

```swift
func ZetaAlgorithm(ptrn: String) -> [Int]? {

    let pattern = Array(ptrn.characters)
    let patternLength: Int = pattern.count

    guard patternLength > 0 else {
        return nil
    }

    var zeta: [Int] = [Int](repeating: 0, count: patternLength)

    var left: Int = 0
    var right: Int = 0
    var k_1: Int = 0
    var betaLength: Int = 0
    var textIndex: Int = 0
    var patternIndex: Int = 0

    for k in 1 ..< patternLength {
        if k > right {  //在 Z-box 之外: 比较字符串直到不匹配
            patternIndex = 0

            while k + patternIndex < patternLength  &&
                pattern[k + patternIndex] == pattern[patternIndex] {
                patternIndex = patternIndex + 1
            }

            zeta[k] = patternIndex

            if zeta[k] > 0 {
                left = k
                right = k + zeta[k] - 1
            }
        } else {  // 在 Z-box 中
            k_1 = k - left + 1
            betaLength = right - k + 1

            if zeta[k_1 - 1] < betaLength { // 全部在 Z-box 中： 可以使用之前计算过的
                zeta[k] = zeta[k_1 - 1]
            } else if zeta[k_1 - 1] >= betaLength { // 不全在 Z-box 中： 必须处理一些比较
                textIndex = betaLength
                patternIndex = right + 1

                while patternIndex < patternLength && pattern[textIndex] == pattern[patternIndex] {
                    textIndex = textIndex + 1
                    patternIndex = patternIndex + 1
                }

                zeta[k] = patternIndex - k
                left = k
                right = patternIndex - 1
            }
        }
    }
    return zeta
}
```

让我们举例说明上面代码。假设 `P = “abababbb” ` 。算法从 `k = 1` 开始， `left = right = 0` 。因此没有“激活” Z-box ，又因为 `k > right` 开始比较 `P[1]` 和 `p[0]`。


       01234567
    k:  x
       abababbb
       x
    Z: 00000000
    left:  0
    right: 0

由于第一次比较后以 `P[1]` 开始的字符串与 `P` 前缀不匹配，因此 `z[1] = 0`，`left` 和 `right` 也没动。开始继续下去 `k = 2`，我们有 `2 > 0` ，因此继续比较 `P[2]` 和 `P[0]`。这次字符匹配了，继续比较直到不匹配。在第 `6` 的位置发生不匹配，相匹配的字符共有 `4` 个，因此 `Z[2] = 4`，`left = k = 2` , `right = k + Z[k] - 1 = 5`。现在有了第一个 Z-box, 字符串为 `"abab"`(注意与 `P` 的前缀相匹配) ，以 `left = 2` 开始。

       01234567
    k:   x
       abababbb
       x
    Z: 00400000
    left:  2
    right: 5

开始处理 `k = 3`。因此 `3 < = 5` ，因此在之前计算的 Z-box 中并也是 `P` 的前缀的一部分。因此看一下之前计算的结果， `k_1 = k - left = 1` 是前面 与`P[k]` 相同的字符，`Z[1] = 0` 并且 `0 < (right - k + 1 = 3)`，所以一定是在 Z-box 范围内，可以直接使用之前计算的值。令 `Z[3] = Z[1] = 0`，`left` 和 `right` 保持不变。

计算 `k = 4` 会执行 外层 `if` 的 `else` 逻辑分支。在内部 `if` 位置 `k_1 = 2` 且 `(Z[2] = 4) >= 5 - 4 + 1` 。因此子字符 `	P[k...r]` 与 `P` 的前 `right - k + 1 = 2` 个字符匹配，但是后面的并不知道。所以必须继续比较从 `r + 1 = 6` 位置字符和`right - k + 1 = 2` 位置的字符。 由于 `P[6] != P[2]`, 所以结果为 `Z[k] = 6 - 4 = 2`, `left = 4`， `right = 5`。

       01234567
    k:     x
       abababbb
       x
    Z: 00402000
    left:  4
    right: 5

循环到 `k = 5`,因为 `k <= right` ， `(Z[k_1] = 0) < (right - k + 1 = 1)`, 结果 `Z[k] = 0`。 继续循环 `6` 和 `7`，执行外层 `if` 的第一个分支，但是都是不匹配，但是 算法得到的 Z-数组 为 `Z = [0, 0, 4, 0, 2, 0, 0, 0]`。

Z-Algorithm 算法是线性时间复杂度，进一步说，Z-Algorithm 计算长度为 `n` 的字符串 `P` 时间复杂度为 `O(n)`。

字符串预处理Z-Algorithm 算法实现在 [ZAlgorithm.swift](./ZAlgorithm.swift) 文件中。

### Z-Algorithm 字符串搜索算法 

上面讨论的 Z-Algorithm 是最简单的线性时间复杂度的字符串匹配算法。只需要将模式串 `P` 和 文本 `T` 连接到一个字符中 `S = P$T`， 这里 `$` 是一个不在 `P` 或者 `T` 中的字符。用上面的算法计算 `S` 得到 Z 数组。现在只需要遍历一下 Z 找到等于 `n` （模式串长度）的元素。如果找到了就算找到了。

```swift
extension String {

    func indexesOf(pattern: String) -> [Int]? {
        let patternLength: Int = pattern.characters.count
        /* 用 Z-Algorithm 计算模式串和文本连接后的字符串 */
        let zeta = ZetaAlgorithm(ptrn: pattern + "💲" + self)

        guard zeta != nil else {
            return nil
        }

        var indexes: [Int] = [Int]()

        /* 遍历 zeta 数组尝试找匹配的模式串 */
        for i in 0 ..< zeta!.count {
            if zeta![i] == patternLength {
                indexes.append(i - patternLength - 1)
            }
        }

        guard !indexes.isEmpty else {
            return nil
        }

        return indexes
    }
}
```

举个例子吧，令 `P = “CATA”` ，`T = "GAGAACATACATGACCAT"` 作为模式串和待查文本。把他们用 `$` 连接起来，得到 `S =  "CATA$GAGAACATACATGACCAT"`。用算法计算后得到如下结果：

                1         2
      01234567890123456789012
      CATA$GAGAACATACATGACCAT
    Z 00000000004000300001300
                ^

遍历 Z 数组在 `10` 的位置我们找到 `Z[10] = 4 = n`。因此可以认为在文本 `10 - n - 1 = 5` 的位置找到了匹配的字符串。

正如之前说的那样，这个算法复杂度是线性的。定义`n` 和 `m` 作为模式串和文本的长度。最后的得到的复杂度为 `O(n + m + 1) = O(n + m)`。

声明：本代码基于1997年剑桥大学出版社 Dan Gusfield 编写的[ "Algorithm on String, Trees and Sequences: Computer Science and Computational Biology"](https://books.google.it/books/about/Algorithms_on_Strings_Trees_and_Sequence.html?id=Ofw5w1yuD8kC&redir_esc=y)  手册。

**作者 Matteo Dunnhofer ，译者 [KeithMorning](https://github.com/KeithMorning/swift-algorithm-club-cn)**
