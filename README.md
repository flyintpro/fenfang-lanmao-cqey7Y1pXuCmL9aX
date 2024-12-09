
题目：给你一个字符串 s，找到 s 中最长的回文子串。


![](https://img2024.cnblogs.com/blog/386841/202412/386841-20241208224934690-300321696.png)


这一题作为中等难度，常规解法对于大多数人应该都没有难度。但是其中也有超难的解决办法，下面我们就一起由易到难，循序渐进地来解这道题。


# ***01***、暴力破解法


对于大多数题目来说，在不考虑性能的情况下，暴力破解法常常是最符合人的思维习惯的。


比如这道题，求一个字符串中最长的回文子串，那么我们只需要把字符串中所有可能的子字符串都判断一下是不是回文串，并找出长度最长的不就行了嘛。


这里需要三层循环，第一层和第二层循环组织出所有可能的子字符串，第三层循环判断是否为回文串。


而判断一个字符串是否为回文串，也很简单，只需要从字符串两端开始判断首尾字符是否相等，如果相等继续向字符串中心方向前进继续比较下一个首尾字符是否相等，直到比较完所有字符，如果都相等则为回文串。其核心思想是由外向内，逐一比较。


![](https://img2024.cnblogs.com/blog/386841/202412/386841-20241208224944204-83754843.png)


具体代码如下：



```
 
//暴力破解法
public static string BruteForce(string s)
{
    var result = string.Empty;
    var max = 0;
    var len = s.Length;
    //从第一个字符开始遍历，作为起始字符
    for (var i = 0; i < len; i++)
    {
        //从第二个字符来开始遍历，作为结束字符
        for (var j = i + 1; j <= len; j++)
        {
            //取出[i,j)字符串,即包含i，不好含j，为临时字符串
            var temp = s[i..j];
            //如果临时字符串是回文字符串，且临时字符串长度大于目标字符串长度
            if (IsPalindromic(temp) && temp.Length > result.Length)
            {
                //则更新目标字符串为当前临时字符串
                result = temp;
                max = Math.Max(max, j - i - 1);
            }
        }
    }
    return result;
}
//判断字符串是否为回文串
public static bool IsPalindromic(string s)
{
    var len = s.Length;
    //遍历字符串的一半长度
    for (var i = 0; i < len / 2; i++)
    {
        //如果对称位置不同，则不为回文串
        if (s[i] != s[len - i - 1])
        {
            return false;
        }
    }
    return true;
}

```

时间复杂度：两层for循环O(n^2），for 循环里边判断是否为回文串O(n），所以时间复杂度为O(n^3）。


空间复杂度：O(1），常数个变量。


# ***02***、暴力破解法优化（动态规划法）


要想优化暴力破解法，我们要先找到它到底有什么问题。它的时间复杂度之所以这么高，是因为有大量的重复计算，可能文字描述不够直观，下面我们先用二维表展示一个字符串的所有子字符串的组合情况，然后再在这个表中看判断是否为回文串时哪些子字符串被重复判断。


![](https://img2024.cnblogs.com/blog/386841/202412/386841-20241208224956551-423426394.png)


如上图在字符串abcde中，在判断其子字符串bcd是否是回文串时，作为字符串bcd已经计算过了，同样的其子字符串c，作为字符串也已经计算过了，其他的沿着箭头方向都是表示存在重复计算的地方。


到这里我们的优化方案就有了，我们可以把已经计算过的存下来，这样下次用到的时候直接拿过来用而不用再计算了。


既然我们通过图就发现了一些规律，我们不妨再深入思考一下，为什么会这样？


如果我们基于暴力破解法中判断是否为回文串的算法定义回文串，那么可得：


P(i,j)\=P(i\+1,j\-1\)\&\&S\[i]\=\=S\[j]


可以理解为如果一个字符串是回文串，那么去掉首尾字符后子字符串依然是回文串。反过来如果子字符串是回文串并且其首尾一个字符相等，那么这个字符串整体也是回文串。


我们再对上图斜对角上加些辅助线，如果我们按所有子字符串的长度分类，则会发现长度为3的依赖长度为1的，长度为5的依赖长度为3的，长度为4的依赖长度为2的。如下图：


![](https://img2024.cnblogs.com/blog/386841/202412/386841-20241208225003880-767857433.png)


长度为1本身就是回文串，长度为2的如果两个字符相等则为回文串，那么所有长度大于等于3的都可以通过长度为1和2的计算出来。


到这里整个算法思路就出来了：先计算长度为1和2的子字符串并存入二维数组，然后基于此二维数组继续计算长度为3、4、5……。


具体代码如下：



```
//动态规划
public static string DynamicProgramming(string s)
{
    var length = s.Length;
    //判断该组合是否是回文字符串，行为起始点，列为结尾点
    var dp = new bool[length, length];
    //最长回文字符串，初始为0
    var result = string.Empty;
    //从回文长度为1开始判断，到字符长度n为止
    for (var len = 1; len <= length; len++)
    {
        for (var startIndex = 0; startIndex < length; startIndex++)
        {
            //结束索引 = 起始索引 + 间隔（len - 1）
            var endIndex = startIndex + len - 1;
            //结束索引超出字符串长度，结束本次循环
            if (endIndex >= length)
            {
                break;
            }
            //回文字符串的公式就是子字符串也是回文，并且当前起始字符和结束字符相等，
            //所以得出公式 dp[startIndex+1,endIndex-1] && s[startIndex] == s[endIndex]
            //其中回文长度为1和2两种特殊情况需要单独处理，其特殊性在于他们不存在子字符串
            //回文长度为1时，自身当然等于自身
            //回文长度为2时，起始字符和结束字符是相邻的，只要相邻的字符相等就可以
            dp[startIndex, endIndex] = (len == 1 || len == 2 || dp[startIndex + 1, endIndex - 1]) && s[startIndex] == s[endIndex];
            //当前字符串是回文，并且当前回文长度大于最长回文长度时，修改result
            if (dp[startIndex, endIndex] && len > result.Length)
            {
                result = s.Substring(startIndex, len);
            }
        }
    }
    return result;
}

```

时间复杂度：O(n^2），即为两层for循环组成的所有子字符串情况。


空间复杂度：O(n^2\)，即存储已计算子字符串结果需要的空间。


这个算法还有一个专有名称：动态规划，我们这里之所以没有突出去讲，是想把整个解题思路展现出来，掌握好了基础解题能力，我们才能做总结，而这个解法的总结就可以概括为动态规划，这是一种通用的思想，后面会经常用到。


# ***03***、中心扩展法


上面的优化虽然时间复杂度降下来了，但是空间复杂度上升了，我们继续想想有什么其他方法呢？


上面的方法判断是否为回文串的核心思想是由外向内，那我们是否可以换个思路——从内向外呢？这就是中心扩展法的核心思想。


在暴力破解法中，我们需要两层循环表示任何一种子字符串排列，而且中心扩展法核心思想是通过一个中心点向两边扩展也就天然只需要一层循环即可表示探测所有字符的情况。


![](https://img2024.cnblogs.com/blog/386841/202412/386841-20241208225015382-246269230.png)


由中心往两边扩展是对称的，而回文串长度可以是奇数也可以是偶数，因此我们在中心扩展时就需要分奇偶两种情况来处理。


![](https://img2024.cnblogs.com/blog/386841/202412/386841-20241208225022229-32612079.png)


到这里我们可以大致梳理一下整体逻辑了：


（1）依次循环处理字符串的每个字符，向两边扩展；


（2）每次扩展分奇偶两种情况处理；


（3）计算出最大长度并保留；


（4）重复（2）、（3）直至所有字符处理完成；


具体实现代码如下：



```
//中心扩散法
public static string CenterExpand(string s)
{
    //如果字符串为空或只有一个字符，直接返回该字符串
    if (s == null || s.Length < 1)
    {
        return "";
    }
    //记录最长回文子串的起始位置和结束位置
    var startIndex = 0;
    var endIndex = 0;
    //遍历每个字符，同时处理回文字串长度为奇偶的情况，
    //即以该字符或该字符与其下一个字符之间为中心的回文
    for (var i = 0; i < s.Length; i++)
    {
        //获取回文字串长度为奇数的情况，
        //即以当前字符为中心的回文长度
        var oddLength = PalindromicLength(s, i, i);
        //获取回文字串长度为偶数的情况，
        //即以当前字符和下一个字符之间的空隙为中心的回文长度
        var evenLength = PalindromicLength(s, i, i + 1);
        //取两种情况下的最长长度
        var maxLength = Math.Max(oddLength, evenLength);
        //如果找到更长的回文子串，更新起始位置和长度
        if (maxLength > endIndex - startIndex)
        {
            //重新计算起始位置
            startIndex = i - (maxLength - 1) / 2;
            //重新计算结束位置
            endIndex = i + maxLength / 2;
        }
    }
    //返回最长回文子串
    return s[startIndex..(endIndex + 1)];
}
//从中心向外扩展，检查并返回回文串的长度
public static int PalindromicLength(string s, int leftIndex, int rightIndex)
{
    //左边界大于等于首字符，右边界小于等于尾字符，并且左右字符相等
    while (leftIndex >= 0 && rightIndex < s.Length && s[leftIndex] == s[rightIndex])
    {
        //从中心往两端扩展一位
        //向左扩展
        --leftIndex;
        //向右扩展
        ++rightIndex;
    }
    //返回回文串的长度（注意本来应该是rightIndex - leftIndex + 1，
    //但是满足条件后leftIndex、rightIndex又分别向左和右又各扩展了一位，
    //因此需要把这两位减掉，所以最后公式为rightIndex - leftIndex - 1）  
    return rightIndex - leftIndex - 1;
}

```

时间复杂度：O(n^2\)。


空间复杂度：O(1\)。


此方法虽然时间复杂度没变，但是空间复杂度大大降低。这也给了我们继续优化的动力。


下一章节我们将详细讲解次题的马拉车解法。


***注***：测试方法代码以及示例源码都已经上传至代码库，有兴趣的可以看看。[https://gitee.com/hugogoos/Planner](https://github.com)


 本博客参考[飞数机场](https://ze16.com)。转载请注明出处！
