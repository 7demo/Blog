# 剑指

## 基本概念

#### 数组

数组与字符串是连续内存，要预先分配，空间性不好，因次有动态数组，每次分配较小内存，不足时，重新分配一块大的空间，原内存空间释放，但是比较耗时。

因为内存连续的原因，数组的读取是O(1),


## 算法

#### 3. 找到数组重复的数字

> 3.0 在长度为n的数组中，所有值都在0 ~ n-1间

*方案一* 先排序，再比较相邻是否一样，时间复杂度为O(nlogn)

*方案二* 利用哈希表，复杂度为O(n), 但是空间复杂度为O(n)

*方案三* 如果数组不存在重复、且有序的情况下，数组中的值必定和该值得索引相同，如果不同则拿该值与以该值为索引的值进行比较，如果相同则为重复值，如果不同，则移动该值到对应的索引位置。空间复杂度为O(1), 时间复杂度为O(n)

```javaScript
function getRepeat(arr, start = 0) {
    let ret = []
    for (let i = start; i < arr.length; i++) {
        // 当前索引值
        let value = arr[i]
        // 以当前索引值为索引的 值
        let valueByvalue = arr[value]
        // 相同，则是有序不重复数组，则无问题
        // 不同 则若两者一样，则需要移动位置，重新计算比较当前索引的值
        if (value !== i) {
            if (value === valueByvalue) {
                ret.push(value)
            } else {
                arr[i] = valueByvalue
                arr[value] = value
                i--
            }
        }
    }
    return ret
}
```

当不能修改源数组时，改如何操作？

> 3.1 长度为n+1的数组，值都在1~n范围内。

*方案一* 复制数组如上操作，但是空间复杂度为O(n).

*方案二* 如果避免使用辅助空间，如果长度为n+1, 取其中间数m，分为大于m的一组，小于m的一组，如果切分的数组的长度超过了m至另外一端的值，那就是有重复的。空间复杂度为O(1), 时间复杂度为O(nlogn)

```javaScript
// 遍历整个数组，获得start-end之间元素的个数
function getCount(arr, len, start, end) {
    let count = 0
    for (i = 0; i < len; i++) {
        if (start <= arr[i] && arr[i] <= end) {
            count ++
        }
    }
    return count
}
function getRepeat(arr) {
    let end = arr.length - 1
    let len = arr.length
    let start = 1
    while (end >= start ) {
        let mid = Math.floor((end + start) / 2)
        // 拿到区间元素个数进行比较，如果大于中间值，则必有重复
        // 否则另外一个区间有重复
        // 区间为二分查找，而每次又需要遍历寻找个数，所以为logn * n
        let count = getCount(arr, len, start, mid)
        if (end == start) {
            if (count) {
                return count
            }
        }
        if (count > mid - start + 1) {
            end = mid
        } else {
            start = mid + 1
        }
    }
    return ret
}
```

#### 4. 二维数组是否包含给予的整数

> 二维数组每行都是从左到右递增，从上到下也是递增

```
1 2 8 9
2 4 9 12
4 7 10 13
6 8 11 15
```
二维数组可以视为一个方格，第一次取右上角的值，如果比目标值大，说明最有一列直接可以舍弃。如果比目标值小，说明要从第二行查找。

```javaScript
function find(arr, num) {
    let i = 0
    let j = arr[0].length - 1
    while (i < arr.length && j >= 0) {
        if (arr[i][j] === num) {
            return true
        }
        if (arr[i][j] > num) {
            j --
        } else {
            i++
        }
    }
    return false
}
```

#### 4. 替换字符串

> 替换字符串中的值后，因为新加入的值会移动(重新分配内存)，要求复杂度为O(n)——如果简单遍历移动，替换的为O(n), 移动也为O(n), 最终的复杂度为O(n^2)

```javaScript
// jsdaim
function move(str) {
    let len = str.length
    let replaceEleLen = 0
    for (i = 0; i < len; i++) {
        if (str[i] == ' ') {
            replaceEleLen ++
        }
    }
    let toltalLen = len + replaceEleLen * 2
    while (toltalLen && len) {
        if (!str[toltalLen - 1]) {
            if (str[len - 1]) {
                str[toltalLen - 1] = str[len - 1]
                str[len - 1] = ''
                toltalLen --
            } else {
                str[toltalLen - 1] = '%%'
                toltalLen = toltalLen - 2
            }
        }
        len --
    }
    return str
}
```

#### 7.重建二叉树

> 按照深度优先的情况下，给定前序与中序顺序，由于中序顺序下，第一个值肯定是根节点，并且在中序遍历的时，根节点前的值都在左树，根节点有的值都在右树。

```javaScript
let arr1 = [1,2,4,7,3,5,6,8] // 前序
let arr2 = [4,7,2,1,5,3,8,6] // 中序
function Node(key) {
    this.key = key
    this.left = null
    this.right = null
}
function createTree(pre, mid) {
    let rootKey = pre[0]
    if (!rootKey) {
        return null
    }
    let rootIndex = arr2.indexOf(rootKey)
    let tree = new Node(rootKey)
    let midLeftArr = mid.slice(0, rootIndex)
    let midrightArr = mid.slice(rootIndex + 1)
    let preLeftArr = pre.slice(1, midLeftArr.length + 1)
    let preRightArr = pre.slice(midLeftArr.length + 1)
    tree.left = createTree(preLeftArr, midLeftArr)
    tree.right = createTree(preRightArr, midrightArr)
    return tree
}
```

#### 10.斐波那锲数列

> 最基本的算法如下，但是时间复杂度为O(2^n)——类似一个完全二叉树求值，这个二叉树的高度为n-1，那么节点最多可以有2^n -1 个。

```javaScript
function fib(n) {
    if (n == 0) {
        return 1
    }
    if (n == 1) {
        return 1
    }
    return fib(n-1) + fib(n-2)
}
```

以上算法，采用尾递归，时间复杂度为O(n)

```javaScript
function fib(n, f0=1, f1=1) {
    if (n <= 1) {
        return f1
    }
    return fib(n-1, f1, f1+f0)
}
```

以下是循环法，时间复杂度也为O(n)

```javaScript
function fib(n) {
    let f0 = 1
    let f1 = 1
    let fn = 0
    for (let i = 2; i <= n; i++) {
        fn = f0+f1
        f0 = f1
        f1 = fn
    }
    return fn
}
```

#### 11.旋转数组的最小数字。

> 旋转数组是指把数组的一部分挪到尾部。题目的意思是寻求有序旋转数组的最小值，O(n)是不合格的。此处用二分法，O(logn)

> 开始设置两个指针，分别指向数组的第一个元素与最后一个元素，若中间元素大于最后一个值，则整个数组的最小值就在后一部分，若小于，则位于前面。

```javaScript
function sort(arr) {
    let i = 0
    let j = arr.length - 1
    let m = 0
    while (i + 1 < j) {
        let mid = Math.floor((i+j)/2)
        if (arr[mid] > arr[j]) {
            i = mid
        } else if (arr[mid] < arr[j]) {
            j = mid
        }
    }
    m = Math.min(arr[i], arr[j])
    return m
}
```

#### 回溯法

> 12.矩阵路径。矩阵中是否包含一条路径包含某些节点的路径。节点不可重复。回溯法。比如以下，求必须包含bfce的路径。

a   b   t   g

c   f   c   s

j   d   e   h

```JavaScript
let m = [
    ['a', 'b' ,'t', 'g'],
    ['c', 'f', 'c', 's'],
    ['j', 'd', 'e', 'h']
]

// let str = 'ak'
let str = 'bfce'

function getPath(m, path) {
    let x = 0
    let y = 0
    let xEnd = m[0].length
    let yEnd = m.length
    let visited = [] // 初始化标记组 用来记录已访记录
    let pathLen = 0

    for (let i =0 ; i < xEnd; i++) {
        visited[i] = []
        for (let j = 0; j < yEnd; j++) {
            visited[i][j] = false
        }
    }

    for (; x < xEnd; x++) {
        for (; y < yEnd; y++) {
            if (hasPathCore(m, x, y, xEnd, yEnd, visited, pathLen, path)) {
                return true
            }
        }
    }

    return false
}

function hasPathCore(m, x, y, xEnd, yEnd, visited, pathLen, path) {
    if (path[pathLen] == undefined) {
        return true
    }
    let flag = false
    if (x >= 0 && x < xEnd && y >=0 && y < yEnd && m[x][y] == path[pathLen] && !visited[x][y]) {
        visited[x][y] = true
        pathLen++
        flag = hasPathCore(m, x + 1, y, xEnd, yEnd, visited, pathLen, path) || hasPathCore(m, x - 1, y, xEnd, yEnd, visited, pathLen, path) || hasPathCore(m, x, y + 1, xEnd, yEnd, visited, pathLen, path) || hasPathCore(m, x, y - 1, xEnd, yEnd, visited, pathLen, path)
        if (!flag) {
            pathLen --
            visited[x][y] = false
        }
    }
    return flag
}
```

> 12.2 机器人路径。地上有一个m行n列的方格，一个机器人从坐标（0，0）的位置开始移动，他每次可以向左，右，上，下移动一格，但不能进入行坐标与列坐标数位之和大于K的格子。例如当k=18，机器人可以进入方格为（35,37）,因为3+5+3+7 = 18 ; 而不能进入(36,37) 的格子，因为3+6+3+7 = 19 > 18 ;请问该机器人能够到达多少个格子。

> 依然是回溯法。关于K的判定其实就是坐标分别拆分成单个数字进行相加。

```javascript
function getCount(num, m, n) {
    // 记录是否访问过
    let visited = []
    for (let i = 0; i < m; i++) {
        visited[i] = []
        for (let j = 0; j < n; j++) {
            visited[i][j] = false
        }
    }
    return helper(num, 0, 0, m, n, visited)
}

function helper(num, i, j, m, n, visited) {
    let count = 0
    if (i >= 0 && j >=0 && i < m && j < n && !visited[i][j] && checkPointer(i, j) < num ) {
        visited[i][j] = true
        count ++
        count += helper(num, i + 1, j, m, n, visited) + helper(num, i - 1, j, m, n, visited) + helper(num, i, j + 1, m, n, visited) + helper(num, i, j - 1, m, n, visited)
    }
    return count
}

function checkPointer(i， j) {
    let str = j + '' + i
    let num = 0
    for (let m = 0; m < str.length; i++) {
        num += str[m] / 1
    }
    return num
}
```

> 12.2 剪绳子

> 给你一根长度为n的绳子，请把绳子剪成m段（m、n都是整数，n>1并且m>1），每段绳子的长度记为k[0],k[1],...,k[m]。请问k[0]xk[1]x...xk[m]可能的最大乘积是多少？ 题目中表示必须减一刀。

> 动态规划。在如果在第i的地方剪一刀，则变成fn(i) * fn(n-i)，求各自对大的值。但是需要注意的时，长度为4时，可以分为2*2，但是因为长度为2时积为1，所以4以下的值要特别处理。

```javaScript
function getArea(n) {
    if (n < 2) {
        return 0
    }
    if (n == 2) {
        return 1
    }
    if (n == 3) {
        return 2
    }
    // 需要注意的时，长度为1时，因为必须切一刀，所以此时应该为0*1=0，所以当n大于一定值得时候，在切分的过程中因为参与运算积的原因，为1的时候，必须为1。
    // 同理长度为2，正常来说为1，但是因为参与计算，所以为2。3的为3. 等到4的时候，就可以分解为2*2，就没有必要继续固定了。
    let maxArea = [0, 1, 2, 3]
    // 用来缓存参与计算时对应的切分段落值
    let tmp = [[0],[1],[2],[3]]
    for (let i = 4; i <= n; i++) {
        maxArea[i] = 0
        for (let j = 1; j <= i/2; j++) {
            let area = maxArea[j] * maxArea[i - j]
            if (area > maxArea[i]) {
                maxArea[i] = area
                tmp[i] = [...tmp[j], ...tmp[i - j]]
            }
        }
    }
    console.log(tmp[n])
    return maxArea[n]
}
```

> 贪婪算法。据上动态算法可知，大于3的时候，尽量分为长度为3的段。

```javaScript
function LineMax(n) {
    if (n < 2) {
        return 0
    }
    if (n == 2) {
        return 1
    }
    if (n == 3) {
        return 2
    }
    let max = 1
    while (n > 3) {
        n = n - 3
        max = max * 3
    }
    return max * n
}
```
