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

> 贪婪算法。据上动态算法可知，大于3的时候，尽量分为长度为3的段

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
    while (n >= 5) {
        n = n - 3
        max = max * 3
    }
    if (n == 4) {
        max = max * 2 * 2
    } else {
        max = max * n
    }
    return max
}
```

> 19 正则表达式匹配

> 主要是以模式中第二个值是否为*为作为切入点。

```javaScript
function matchStr(str, mode) {
    if (!str) {
        if (!mode) {
            return true
        } else {
            let index = 0
            while (index < mode.length - 1) {
                if (mode[index + 1] == '*') {
                    index= index + 2
                } else {
                    return false
                }
            }
            return true
        }
    }
    if (!mode && str) {
        return false
    }
    let strIndex = 0
    let modeIndex = 0
    while (strIndex < str.length && modeIndex < mode.length) {
        if (mode[modeIndex + 1] == '*') {
            if (str[strIndex] == mode[modeIndex]) {
                strIndex++
            } else {
                strIndex++
                modeIndex = modeIndex+2
            }
        } else {
            if (str[strIndex] == mode[modeIndex]) {
                strIndex++
                modeIndex++
            } else {
                return false
            }
        }
    }
    return true
}
```

> 20 判断一个字符串是否一个数值

> 特殊情况：第一位有正负号；遇到指数情况，指数的值必须大于小数点距离e的距离。

> 思路：一个字符串分为三个部分：整数、小数、指数。

```javaScript
 /* 正负号只能有一个且只能在第一位
    * 指数后面必须有数字
    * .23 视为小数
     需要注意的是 字符串与字符串相比较会两个先转assci码进行比较。而字符串与数字比较只有字符串转assci。
    */
function check(str) {
    if (!str && str !== '0') {
        return false
    }
    let pointIndex = -1
    let exIndex = -1
    let sign = -1
    for (let i = 0; i < str.length; i++) {
        if (str[i] == '+' || str[i] == '-') {
            exIndex = i
        } else if (str[i] == 'E' || str[i] == 'e') {
            if (exIndex > -1) {
                return false
            }
            if (str[i + 1] < 0 || str[i + 1] > 9 || !str[i + 1] ) {
                return false
            } else {
                exIndex = i
            }
        } else if (str[i] == '.') {
            if (pointIndex > -1) {
                return false
            }
            pointIndex = i
        } else if (str[i] < 0 || str[i] > 9) {
            return false
        } else {
            continue
        }
    }
    return true
}
```

> 21. 调整数组顺序使奇数位于偶数前面

```javaScript
function sort(arr, helper) {
    let start = 0
    let end = arr.length - 1
    while (start < end) {
        if (helper(arr[start])) {
            if (helper(arr[end])) {
                end --
            } else {
                let tmp = arr[start]
                arr[start] = arr[end]
                arr[end] = tmp
                start++
            }
        } else {
            start++
        }
    }
    return arr
}
function helper(nun) {
    return (nun & 1) == 0
}
```

> 22. 链表中的倒数第K个节点

> 第一个简单思路就是，第一次遍历链表得到链表的长度n，那么倒数k就是n-k+1。但是这样会循环遍历两次。

> 第二个思路：有两个指针，开始循环时，一个指针正常遍历，直到到k-1位置，此时两个指针的间距为k。然后下次遍历，两个指针则一起递增，直至第一个指针遍历结束。

```javaScript
function Node (val) {
    this.val = val
    this.next = null
}
function List() {
    this.list = null
}
List.prototype.append = function (num) {
    if (!this.list) {
        this.list = new Node(num)
    } else {
        let tmp = this.list
        while (tmp.next) {
            tmp = tmp.next
        }
        tmp.next = new Node(num)
    }
}

function getK(list, k) {
    let index1 = 0
    let index2 = 0
    let ret = list
    while (list && list.next) {
        if (index2 == index1 + k - 1) {
            index1++
            ret = ret.next
        }
        index2++
        list = list.next
    }
    return ret

}
let list = new List()
for (let i = 0 ; i < 9; i++) {
    list.append(i)
}
```

> 22.1 链表如果为奇数，则返回中间那个，如果为偶数，则返回中间的任意一个。

> 同样，采用双指针，一个指针每次走一步，而另外一个指针走两步。

```javaScript
function getM(list) {
    let ret1 = list
    let ret2 = list
    while (ret2 && ret2.next) {
        ret2 = ret2.next
        if (ret2) {
            ret2 = ret2.next
        }
        ret1 = ret1.next
    }
    return ret1

}
```

> 23. 寻找链表中的入口环点

> 在限制空间的情况下，确定是否有环：两个指针，从开始一个指针每次走一步，另外一个指针每次走两步。如果走得快的指针赶上或者超过。那么就是有环存在。具体：

> 1. 设定两个指针分别为slow与fast，一个速度为1，一个速度为2。在存在环点x的情况下，假设环的周长为c。两个指针相遇在b点。假设从开始到x点的长度为x， 从x点到b点的长为b。

> 2. 相遇时，fast走了x+nc+b，而slow走了x+b。因为fast是slow速度的两倍，x+nc+b = 2*(x+b)，得到x=nc-b。这里的nc-b就是环的n个周长减去b的长度，也是fast从b点走nc-b的长度，此时如果slow从0开始走到x点，两个则必会相遇到x点。注意此时，因为长度一致，所以此时走路速度是一样的。

```javaScript
function getX(list) {
    let slow = list
    let fast = list
    while (true) {
        slow = slow.next
        fast = fast.next.next
        if (slow.val == fast.val) {
            break
        }
    }
    slow = list
    while (slow.val != fast.val) {
        slow = slow.next
        fast = fast.next
    }
    return slow
}
```

> 24. 翻转链表

> 不采用堆栈，采用断开下个连接，调整当前与之前的关系。

```javaScript
function trunList(list){
    let pre = null
    let next = list.next
    while (true) {
        list.next = pre
        pre = list
        // 如果next不存在 则到最深，则要跳出循环
        if (next) {
            list = next
            next = next.next
        } else {
            break;
        }
    }
    return list
}
```

> 25. 合并排序链表

```javaScript
function merge(list1, list2){
    // 最终结构
    let ret
    // 保存当前指针节点
    let ex
    // 从两个链表中拿到的节点
    let tmp
    while (list1 || list2) {
        if (list1 && list2) {
            if (list1.val > list2.val) {
                tmp = list2
                list2 = list2.next
            } else {
                tmp = list1
                list1 = list1.next
            }
        } else {
            if (!list1 && list2) {
                tmp = list2
                list2 = list2.next
            } else {
                tmp = list1
                list1 = list1.next
            }
        }
        // 第一个节点
        // 当然指针也是在第一个节点
        if (!ret) {
            ret = tmp
            ex = tmp
        } else {
            // 把当前的指针的next执行新拿到的节点
            ex.next = tmp
            // 重新保存指针节点
            ex = tmp
        }

    }
    return ret
}
```

> 26. 树的子结构

```javaScript
function check(tree1, tree2) {
    if (!tree2) {
        return true
    }
    if (!tree1 && tree2) {
        return false
    }
    let flag
    if (tree1.value == tree2.value) {
        flag = (check(tree1.left,tree2.left) && check(tree1.right,tree2.right)) || check(tree1.left,tree2) || check(tree1.right,tree2)
    } else {
        flag = check(tree1.left,tree2) || check(tree1.right,tree2)
    }
    return flag
}

function check(tree1, tree2) {
    if (!tree2) {
        return true
    }
    if (!tree1 && tree2) {
        return false
    }
    let flag
    if (tree1.value == tree2.value) {
        let flagLeft = check(tree1.left,tree2.left)
        let flagRight = check(tree1.right,tree2.right)
        if (!flagLeft || !flagRight) {
            flagLeft = check(tree1.left,tree2)
        }
        flag = flagLeft || flagRight
    } else {
        flag = check(tree1.left,tree2) || check(tree1.right,tree2)
    }
    return flag
}
```

> 27. 二叉树镜像

```javaScript
function turn(list) {
    if (!list) {
        return list
    }
    let tmp = list.left
    list.left = turn(list.right)
    list.right = turn(tmp)
    return list
}
```

> 28. 对称二叉树

> 对称二叉树是本身与镜像一样。前序遍历二叉树是先根节点再左树再右树，如果这样得到的值与先根节点再右树再左树的节点值一样，那么就是对称二叉树。

```javaScript
function check(list) {
    let tra1 = helperL(list)
    let tra2 = helperR(list)
    if (tra1.join('') == tra2.join('')) {
        return true
    } else {
        return false
    }

}
function helperL(list) {
    if (!list) {
        return []
    }
    return [list.value, ...helperL(list.left), ...helperL(list.right)]
}
function helperR(list) {
    if (!list) {
        return []
    }
    return [list.value, ...helperR(list.right), ...helperR(list.left)]
}
// 因为如果相对称的话，左树的值与右树的值必定是一样对称的。所以左树按照正常的前序遍历，而右树按照定制的前序遍历，对比即可。整体少遍历一遍。

function check1(list) {
    let left = helperL(list.left)
    let right = helperR(list.right)
    if (left.join('') == right.join('')) {
        return true
    } else {
        return false
    }
}
```

> 29.顺时针打印矩阵

```javaScript
function print(arr) {
    // 左上坐标
    // 右下坐标
    let xLen = arr[0].length
    let yLen = arr.length
    let xStart = 0
    let yStart = 0
    // 指针
    let x = 0
    let y = 0
    // 最后的个数
    let num = xLen * yLen
    let ret = []
    // 分别表示当前应该变化是那一边
    let xFlag = true
    let yFlag = false
    let lFlag = false
    let tFlag = false
    while (ret.length < num) {
        ret.push(arr[y][x])
        // 代表x参数变化
        if (xFlag && !yFlag && !lFlag && !tFlag) {
            if (x < xLen - 1) {
                x++
            } else if (x == xLen - 1) {
                xFlag = false
                yFlag = true
                xLen--
                y++
            }
        // 代表y参数变化
        } else if (!xFlag && yFlag && !lFlag && !tFlag) {
            if (y < yLen - 1) {
                y ++
            } else if (y == yLen -1) {
                yFlag = false
                lFlag = true
                yLen--
                x--
            }
        } else if (!xFlag && !yFlag && lFlag && !tFlag) {
            if (x > xStart) {
                x--
            } else if(x == xStart) {
                lFlag = false
                tFlag = true
                yStart++
                y--
            }
        } else if (!xFlag && !yFlag && !lFlag && tFlag) {
            if (y > yStart) {
                y--
            } else if(y == yStart) {
                xFlag = true
                tFlag = false
                x++
                xStart++
                yStart++
            }
        }
    }
    return ret
}
```

> 也可以：1，每次都是以左上角打印一圈，2，递增左上角坐标进行边界控制。

```javaScript
function print(arr) {
    let ret = []
    let start = 0
    let xLen = arr[0].length
    let yLen = arr.length
    while (2*start < xLen && 2*start < yLen) {
        ret.push(...helper(start, xLen, yLen, arr))
        start++
        xLen--
        yLen--
    }
    return ret
}
function helper(start, xLen, yLen, arr) {
    let ret = []
    // 打印左到右
    for (let i = start; i < xLen; i++) {
        ret.push(arr[start][i])
    }
    // 打印从上到下
    // 前提条件是只少有两行
    if (start < yLen) {
        for (let i = start + 1; i < yLen; i++) {
            ret.push(arr[i][xLen - 1])
        }
    }
    // 打印从右到左
    // 前提条件是只少有两行
    if (start < yLen) {
        for (let i = xLen - 2; i >= start; i--) {
            ret.push(arr[yLen - 1][i])
        }
    }

    // 打印从下到上
    // 前提条件是只少有两行
    if (start < yLen) {
        for (let i = yLen - 2; i > start; i--) {
            ret.push(arr[i][start])
        }
    }
    return ret
}
```

> 31.栈的压入与弹出序列。

```javascript
// 其实按照arr1的顺序，压进新栈aim，过程中和期望的出栈顺序进行比较，一次对aim执行出栈操作。最后arr1, aim全为空
function check(arr1, arr2){
    // 按照出栈顺序保存的数组，为了和arr2进行比较
    let tmp = []
    // 辅助栈，为了按照arr1的顺序压进去
    let aim = []
    // 指示该出栈的索引值
    let i = 0
    while (arr1.length || aim.length) {
        // 第一次肯定是进栈
        if (aim.length == 0) {
            aim.push(arr1.shift())
        } else {
            // 如果辅助栈最后一位，就是出栈的一位与期望的顺序不一致，并且arr1仍然有值，则把arr1全部按照顺序放进aim
            if (aim[aim.length - 1] != arr2[i] && arr1.length) {
                aim.push(arr1.shift())
            // 如果aim的最后一位恰好与该出栈的一位一致，则出栈。出栈索引++
            } else {
                tmp.push(aim.pop())
                i++
            }
        }
    }
    return tmp.join('') == arr2.join('')
}
```

> 32.从上到下打印二叉树

```javaScript
function getVal(tree){
    let arr = [tree]
    let ret = []
    let i = 0
    while (i < arr.length) {
        if (Object.prototype.toString.call(arr[i]) == '[object Object]') {
            ret.push(arr[i].value)
            if (arr[i].left !== undefined) {
                arr.push(arr[i].left)
            }
            if (arr[i].right !== undefined) {
                arr.push(arr[i].right)
            }
            i++
        }
    }
    return ret
}
```

> 34.二叉树的和为某一路径的值

```javaScript
function getPath(tree, num){
    let allPath = []
    let path = []
    function helper(tree, num) {
        if (!tree) {
            return
        }
        path.push(tree.value)
        num -= tree.value
        if (tree.left == null && tree.right == null && num == 0) {
            allPath.push([...path])
        }
        helper(tree.left, num)
        helper(tree.right, num)
        path.pop()
    }
    helper(tree, num)
    return allPath
}
```

> 35.复杂链表的复制

```javaScript
// 空间换时间，遍历两次，一次生成链表，同时保存一个hash表来存各个节点。再次遍历指向sib
function clone(list) {
    let obj = {}
    let node
    let tmp // 指向当前指针
    let cur = list
    while (cur) {
        let _node = new Node(cur.value)
        obj[cur.value] = _node
        if (!node) {
            node = _node
            tmp = _node
        } else {
            tmp.next = _node
            tmp = _node
        }
        cur = cur.next
    }
    let nsib = node
    let sib = list
    while (sib) {
        if (sib.sib) {
            nsib.sib = obj[sib.sib.value]
        }
        sib = sib.next
        nsib = nsib.next
    }
    return node
}

// 方法二

function clone(list) {
    let cur = list
    // 每个节点后都复制一个节点进行查 如A-B-C变成A-A`-B-B`-C-C`
    while (cur) {
        let node = new Node(cur.value)
        let tmp  = cur.next
        cur.next = node
        node.next = tmp
        cur = tmp
    }
    // 如果ABC的指向设置A`B`C`的指向（都在其ABC下一级）
    let sib = list
    while(sib) {
        if (sib.sib) {
            let m = sib.sib
            sib.next.sib = m.next
        }
        sib = sib.next.next
    }
    // 拆分链表
    let node = list.next
    let index = list.next
    if (index.next) {
        index = index.next.next
        node.next = index
    }
    return node
}
```

> 36. 二叉树搜索树转成链表

> 中序遍历后，遍历调整指针

```javaScript
function ch(tree) {
    function helper(tree1) {
        let tt = []
        if (!tree1) {
            return []
        }
        tt.push(...helper(tree1.left), tree1, ...helper(tree1.right))
        return tt
    }
    let ret = helper(tree)
    for (let i = 0; i < ret.length; i++) {
        if (ret[i - 1]) {
            ret[i].left = ret[i-1]
        }
        if (ret[i + 1]) {
            ret[i].right = ret[i+1]
        }
    }
    return tree
}

// 处理左树，左树成链表，把左树的最右连接到root
// 处理右树，右树成链表，把右树的最左连接到root
function co(tree) {
    if (!tree) {
        return null
    }
    if (!tree.left && !tree.right) {
        return tree
    }

    let left = co(tree.left)
    let right = co(tree.right)

    let tmp = left
    while (tmp && tmp.right) {
        tmp = tmp.right
    }

    if (tmp) {
        tmp.right = tree
        tree.left = tmp
    }

    if (right) {
        right.left = tree
        tree.right = right
    }
    return left ? left : tree

}
```

> 37.序列化二叉树

> 遍历二叉树时，遍历到叶节点（null）,用特殊符号表示。

```javaScript
function ser(tree) {
    if (!tree) {
        return ['$']
    }
    let ret = []
    ret = [tree.value, ...ser(tree.left), ...ser(tree.right)]
    return ret
}
// [10, 6, 4, "$", "$", "$", 14, 12, "$", "$", 16, "$", "$"]
// 如果数组的第一次数字有值得话，根据结果是前序，所以这个值一定是左树的值。根据函数执行栈的特性，是先构建了10，在构建了10的左树6，在构建6的左树4，再构建4的左树￥，但是此时是$,所以不再增加函数栈，开始出一个栈，此时函数栈的栈顶的函数已经构建了4的左树（停止），再构建4的右树，因为是￥的原因所以不继续增加栈，则4树构建完，继续出栈。此时6的左树已经构建完毕，右树是￥。继续出，到10的左树构建完毕，开始构建右树。。。
function deser(arr) {
    if (!arr || !arr.length) {
        return
    }
    let ret = null
    let value = arr.shift()
    if (value !== '$') {
        ret = {
            value: value
        }
        ret.left = deser(arr)
        ret.right = deser(arr)
    }
    return ret
}
```

> 38.字符串排列

> 38.1 字符串全排列

```javaScript
function print(str) {
    let ret = []
    for (let i = 0; i < str.length; i++) {
        let _str = str.substr(0, i) + str.substr(i + 1)
        let _arr = print(_str)
        if (_arr.length) {
            for (let j = 0; j < _arr.length; j++) {
                ret.push(str[i] + _arr[j])
            }
        } else {
            ret.push(str[i])
        }
    }
    return ret
}
```

> 38.2 字符串不全组合。比如 输入abc，可以打印a,b,c,ab,bc,ac,abc

> 分别打印一位、两位、三位

```javaScript
function print(str) {
    let ret = []
    let step = 1
    while (step <= str.length) {
        for (let i = 0; i <= str.length - step; i++) {
            ret.push(str.substr(i, step))
        }
        step++
    }
    return ret
}
```

> 38.3 输入一个数组，有8个数字，能否放到一个正方体的八个顶点上。即是全排列(38.1)，然后判断是会否满足a1+a2+a3+a4=a5+a6+a7+a8/a1+a3+a5+a7=a8+a2+a4+a6/a1+a5+a6+a2=a3+a4+a7+a8

> 38.4 八皇后问题。

```javaScript
function queue(arr) {
    // 先找到1-8所有不同的组合
    let tmp = print(arr.join(''))
    // 过滤所有在一条对角线的
    let ret = tmp.filter(item => {
        if (check(item)) {
            return false
        }
        return true
    })
    return ret
}
function check(item) {
    let flag = false
    let arr = item.split('')
    for (let i = 0; i < arr.length - 1; i++) {
        for (let j = i+1; j < arr.length; j++) {
            if (Math.abs(j - i) == Math.abs(arr[j] - arr[i])) {
                return true
            }
        }
    }
    return false
}
function print(str) {
    let ret = []
    for (let i = 0; i < str.length; i++) {
        let _str = str.substr(0, i) + str.substr(i + 1)
        let _arr = print(_str)
        if (_arr.length) {
            for (let j = 0; j < _arr.length; j++) {
                ret.push(str[i] + _arr[j])
            }
        } else {
            ret.push(str[i])
        }
    }
    return ret
}
```

> 39.数组中出现超过一半的数字，要求时间复杂度为O(N)。

> 如果直接统计次数，则复杂度为O(N+M)，M是不同元素的个数。

> 思路一：如果此数组是（部分）有序的，因为超过一半数字都是同一个数，所以该数字肯定在数组的中最中间，所以只需求中位数即可。

```javaScript
function partition(arr, i, j) {
    let mid = i
    let base = arr[i]
    while (i <= j) {
        if (arr[i] > base) {
            let tmp = arr[i]
            arr[i] = arr[mid]
            arr[mid] = tmp
            mid++
        }
        i++
    }
    return mid
}
function getNum(arr) {
    let len = arr.length
    let mid = len>>1
    let start = 0
    let end = len - 1
    while (start != mid) {
        let index = partition(arr, start, len)
        if (index == mid || arr[index] == arr[mid]) {
            return arr[mid]
        } else if (index > mid) {
            len = index
        } else {
            start = index
        }
    }
    return arr[mid]
}
```

> 思路二：计数。一个hash表存每个数出现的次数，遍历时对此操作，同时比较是否是出现最多的。

```javaScript
function getNum1(arr) {
    let max = arr[0]
    let map = {}
    for (let i = 0; i < arr.length; i++) {
        if (map[arr[i]] !== undefined) {
            map[arr[i]] ++
        } else {
            map[arr[i]] = 1
        }
        if (map[arr[i]] > map[max]) {
            max = arr[i]
        }

    }
    return max
}
```

> 思路三：如果计出现次数最多的数每个都为1的话，其他数字为-1。那么最后相加则肯定大于0。

```javaScript
function getNum2(arr) {
    let num = 0
    let count = 0
    for (let i = 0; i < arr.length; i++) {
        if (count == 0) {
            num = arr[i]
            count = 1
        } else {
            if (num == arr[i]) {
                count++
            } else {
                count--
            }
        }
    }
    return num
}
```

> 40. 最小的k个数字。

```javaScript
function par(arr, i, j) {
    let mid = i
    let base = arr[(i+j)>>1]
    while (i <= j) {
        if (arr[i] < base) {
            let tmp = arr[i]
            arr[i] = arr[mid]
            arr[mid] = tmp
            mid++
        }
        i++
    }
    return mid
}
function topK(arr, k) {
    let len = arr.length
    let start = 0
    let end = len - 1

    while (start <= len) {
        let index = par(arr, start, len)
        if (index == k) {
            return arr.slice(0,k)
        } else if (index > k) {
            len = index
        } else {
            start = index
        }
    }
    return undefined
}
```

> 如果不能修改数组，则是堆+遍历的形式。

> 41.数据流中的中位数。

> 如果是一定量的数据个数是奇数，那么中位数是中间那个数，如果是偶数，中位数是中间两数字的平均值。所以在数组变化时，需要知道已经的两个指针，这个指针数组时偶数的时，分别指向中间两个数，左指针比左边的数都打，右指针比右边的数字都小；如果是奇数时，则两个指针是同一个值。终上，紧要的有两点：1，新值插入左侧还是右侧，2，因为数据流动在变，所以插入与查找都要求高效。最终采用大顶堆与小丁堆来实现——js上最大堆的构造复杂度为O(nlogn)。

```javaScript
// 大顶堆 与 小顶堆
function Heap() {
    this.data = []
}
Heap.prototype.insertMin = function(num) {
    this.data.push(num)
    this.checkMmin()
}

Heap.prototype.insertMax = function(num) {
    this.data.push(num)
    this.checkMax()
}

Heap.prototype.checkMax = function() {
    let data = this.data
    for (let i = 1; i < data.length; i++) {
        let cur = i
        let parent = (cur - 1)>>1
        while (data[cur] > data[parent]) {
            swap(data, cur, parent)
            cur = parent
            parent = (cur - 1)>>1
        }
    }
    this.data = data
    return data
}
Heap.prototype.checkMmin = function() {
    let data = this.data
    for (let i = 1; i < data.length; i++) {
        let cur = i
        let parent = (cur - 1)>>1
        while (data[cur] < data[parent]) {
            swap(data, cur, parent)
            cur = parent
            parent = (cur - 1)>>1
        }
    }
    this.data = data
    return data
}
Heap.prototype.pop = function() {
    let num = this.data.shift()
    return num
}
let maxHeap = new Heap()
let minHeap = new Heap()
let count = 0
// 设定奇数时 只插入最大堆
function getMid(num) {
    // 新插入值为奇数
    if ((count & 1) == 0) {
        let min = minHeap.data[0] || 0
        if (num < min || min == 0) {
            maxHeap.insertMax(num)
        } else {
            let _min = minHeap.pop() || 0
            minHeap.insertMin(num)
            maxHeap.insertMax(_min)
        }
    } else {
        let max = minHeap.data[0] || 0
        if (max > num) {
            let _max = maxHeapx.pop()
            minHeap.insertMin(_max)
            maxHeap.insertMax(num)
        } else {
            minHeap.insertMin(num)
        }
    }
    count++
    // 现在整个为偶数
    if ((count & 1) == 0 && count > 1) {
        let min = minHeap.data[0]
        let max = maxHeap.data[0]
        return (min+max)/2
    } else {
        return maxHeap.data[0]
    }
}

```

> 42.连续子数组的最大和。子数组由数组中一个或者连续多个数字组成。

> 要求时间复杂度为O(n)。遍历，开始设置初始值为0，进行累加，如果此时累加值小于0的话，说明要重新开始累加。每次都要比较当前的和是否为最大。

```javaScript
function getV(arr) {
    let max = 0
    let res = arr[0]
    for (let i = 0; i < arr.length; i++) {
        if (max < 0) {
            max = arr[i]
        } else {
            max += arr[i]
        }
        if (max > res) {
            res = max
        }
    }
    return res
}
```

> 动态规划。便利时每次的最大和就是“前一个最大和加上当前值”与“当前值”的大者。

```javaScript

function getVD(arr) {
    if (!arr || !arr.length) {
        return 0
    }
    let len = arr.length
    if (len == 1) {
        return arr[0]
    }
    let ret = []
    ret[0] = arr[0]
    let max = ret[0]
    for (let i = 1; i < len; i++) {
        ret[i] = Math.max(arr[i] + ret[i - 1], arr[i])
        max = Math.max(max, ret[i])
    }
    return max
}
```

> 43. 1-n整数中，1出现的次数。

> 个数数字，相当于1-10中：个位上1出现了一次。

> 十位数字，相当于1-100中：十位上1出现了10次

> 百位数字，相当于1-1000中，百位数1出现了100次

> 所以对于数字n（比如2134），从最右i开始数，i=1。

> 如果第i位的数字大于1，则第i位1的个数为（最高位+1）* 10^i-1（最高位就是从i+1位起到最高位，比如213）

> 如果第i位数字小于1，则i位1的个数为（最高位）*10^i-1

> 如果第i位数字等于1，则位1的个数为 最高位）*10^i-1   +  (低位数+1)。低位数是i-1位直到个数如，在算2134时百位1的个数为2\*10^2+(34+1)

```javaScript
function getNum(num) {
    num = num + ''
    let len = num.length
    let ret = 0
    let i = 1
    while (i <= len) {
        let cur = num[len - i]
        if (cur > 1) {
            let hight = num.substr(0, len - i)
            ret += (hight/1 + 1) * Math.pow(10, i - 1)
        } else if (cur < 1) {
            let hight = num.substr(0, len - i)
            ret += (hight/1) * Math.pow(10, i - 1)
        } else {
            let hight = num.substr(0, len - i)
            let low = num.substr(len - i + 1)
            ret += (hight/1) * Math.pow(10, i - 1) + low/1 + 1
        }
        i++
    }
    return ret
}
```

> 44.数字序列中某一位的数字

> 除了前十位数字外，10-99有9\*10\*2数字，100-999有9\*100\*3数字。相当于两位数有9\*(10^1)\*2个、三位数有9\*(10^2)\*3...即，n位数有9\*(10^n-1\*n)个。

```javaScript
function getNum(num) {
    if (num < 10) {
        return num - 1
    }
    let i = 2
    // 减去个数为
    num = num - 10
    // 如果大于个数小于n位数所占的个数，减去n位数所占的个数，则位数加1
    while (num > 9 * Math.pow(10, i - 1) * i) {
        num -= 9 * Math.pow(10, i - 1) * i
        i++
    }
    // 算出当前位数下，所求值为所占该数的第几个。如num=811, num=3, 那s=1
    let s = num % i
    // count 计算出当前位数i下的个数
    let count = (num - s)/i
    // ret为初始num的真实值
    let ret = count + Math.pow(10, i - 1) + ''
    return ret[s]

}
```

> 45.把数组排成最小的数

> 思路一：根据每个数字的最高位从0-9进行归类，每类再进行大小排序。那么最后遍历生成的值就是最下。复杂度为地一遍遍历O(N) + 第二次遍历O(NlogN) + O（N）

```javaScript
function getNum(arr) {
    let map = {}
    for (let i = 0; i < arr.length; i++) {
        let num = arr[i] + ''
        if (map[num[0]]) {
            map[num[0]].push(arr[i])
        } else {
            map[num[0]] = [arr[i]]
        }
    }
    let ret = ''
    for (let i in map) {
        map[i] = map[i].sort()
        for (let j = 0; j < map[i].length; j++) {
            ret += map[i][j]
        }
    }
    return ret/1
}
```

> 思路二：把数组中的每个数字都变成字符串，然后按照从小到大排序，其实就是转成首位(一次降低)按照ASCII值得比较排序，最后拼成字符串数字即可。比如'13'的ASCII的值是小于'7'的ASCII的值。时间复杂度为O(nlogn)，就是快排。

```javaScript
function sort(arr) {
    if (arr.length < 2) {
        return arr
    }
    let ret = []
    let mid = arr.length >> 1
    let base = arr[mid] + ''
    let left = []
    let right = []
    for (let i = 0; i < arr.length; i++) {
        let num = arr[i] + ''
        if (num < base) {
            left.push(num)
        } else if(num > base) {
            right.push(num)
        }
    }
    left = sort(left)
    right = sort(right)
    ret = [...left, base, ...right]
    return ret
}
function gn(arr) {
    let tmp = sort(arr)
    let ret = ''
    for (let j = 0; j < tmp.length; j++) {
        ret += tmp[j]
    }
    return ret/1
}
```

> 46. 数字翻译成字符串

> 动态规划。比如12258，可以分解为1与2258+12与258.然后继续分解下去

```javaScript
function tran(num) {
    if (!num) {
        return ''
    }
    let dict = 'abcdefjhigklmnopqrstuvwxyz'
    num = num + ''
    let ret = []
    if (num.length < 2) {
        return [dict[num/1]]
    } else {
        let head = num.substr(0, 2) / 1
        if (head < 26) {
            let res = tran(num.substr(1))
            let first = dict[num[0]/1]
            for (let i = 0; i < res.length; i++) {
                ret.push(first+res[i])
            }


            let res2 = tran(num.substr(2))
            let first2 = dict[[num.substr(0,2)]/1]
            for (let i = 0; i < res2.length; i++) {
                ret.push(first2+res2[i])
            }
        } else {
            let res = tran(num.substr(1))
            let first = dict[num[0]/1]
            for (let k = 0; k < res.length; k++) {
                ret.push(first+res[k])
            }
        }
    }
    return ret
}
```

> 47.礼物的最大价值

```javaScript
let arr = [
    [1,10,3,8],
    [12,2,9,6],
    [5,7,4,11],
    [3,7,16,5]
]

function getMax(arr) {
    let ret = arr[0][0]
    let yEnd = arr.length
    let xEnd = arr[0].length
    let x = 0
    let y = 0
    while (x < xEnd - 1 || y < yEnd - 1) {
        if (x + 1 == xEnd) {
            ret += arr[y+1][x]
            y++
            continue
        }

        if (y + 1 == yEnd) {
            ret += arr[y][x+1]
            x++
            continue
        }

        let right = arr[y][x+1]
        let down = arr[y+1][x]
        if (right > down) {
            ret += right
            x++
        } else {
            ret += down
            y++
        }
    }
    return ret
}
```

> 48. 最长不含重复字符串的子字符串。由a-z组成。

```javascript
function getLongStr(str) {
    let position = {}
    let ret = ''
    let tmp = ''
    for (let i = 0; i < str.length; i++) {
        let cur = str[i]
        // 如果当前字母没有出现过，则直接累加
        if (position[cur] === undefined) {
            position[cur] = i
            ret += cur
        } else {
            let prePosition = position[cur]
            // 当前字母距离上次的字母距离大于现在最长字符串的长度，那么说明当前字母上次没有出现在现最长字符串中，则可以直接加
            if (i- prePosition > ret.length) {
                position[cur] = i
                ret += cur
            // 如果当前字母距离上次的字母距离小于长度，说明这个字母在最长的中
            } else {
                position[cur] = i
                ret = str.substr(prePosition, i - prePosition)
            }
        }
        if (tmp.length <= ret.length) {
            tmp = ret
        }
    }
    return tmp
}
```

> 49.丑数。

```javaScript
function getUgly(num) {
    if (num == 0) {
        return 0
    }
    // t2 t3 t5表示数组中 乘以2 、3、5刚好达到最大丑数的位置
    let t2 = t3 = t5 = 0
    let ret = [1]
    let i = 1
    // 每次寻找刚好达到的最大丑数
    // 更新t2 t3  t5的位置
    while (i < num) {
        let m2 = 2 * ret[t2]
        let m3 = 3 * ret[t3]
        let m5 = 5 * ret[t5]
        let min = Math.min(m2, m3, m5)
        ret.push(min)
        while (ret[t2] * 2 <= min) {
            t2++
        }
        while (ret[t3] * 3 <= min) {
            t3++
        }
        while (ret[t5] * 5 <= min) {
            t5++
        }
        i++
    }
    return ret[num - 1]
}
```

> 50.第一个只出现一次的字符

```javaScript
function getFirstLetter(str) {
    let map = {}
    for (let i = 0; i < str.length; i++) {
        let letter = str[i]
        if (map[letter] === undefined) {
            map[letter] = 1
        } else {
            map[letter]++
        }
    }
    for (let i = 0; i < str.length; i++) {
        let letter = str[i]
        if (map[letter] === 1) {
            return letter
        }
    }
    return ''
}
```

> 50.1 判断两个单词是否为变位词

```javaScript
function check(str1, str2) {
    if (str1.length !== str2.length) {
        return false
    }
    let map = {}
    for (let i = 0; i < str1.length; i++) {
        let letter = str[i]
        if (map[letter] === undefined) {
            map[letter] = 1
        } else {
            map[letter]++
        }
    }
    for (let i = 0; i < str2.length; i++) {
        let letter = str[i]
        if (map[letter] !== undefined) {
            map[letter]--
        }
    }
    let flag = true
    for (let i in map) {
        if (map[i] != 0) {
            flag = false
        }
    }
    return flag
}
```

> 51. 数组中的逆序对

> 如果单纯遍历两次，则最终复杂度为O(n^2)。这里可以按照合并排序的方式来处理。比如数组[7，5，6，4]

> 可以把数组分成两个数组，一直分下去只至最后一个处理单位。[7]与[5]，5与7进行比较，可以得到一对符合要求的逆序。同时返回[5,7]，同样6与4也如此操作，又一对符合要求的逆序[4,6]。然后有两个指针，分别指向7与6，比较7与6，因为7大于6，所以可以得到2对，这里2是因为后面的数组[4,6]的长度为2。依次递归。

```javaScript
function check(arr) {
    let ret = 0
    let a = helper(arr)
    function helper(arr) {
        if (!arr || arr.length < 2) {
            return arr
        }
        let tmp = []
        let start = 0
        let end = arr.length
        let mid = (start + end) >> 1
        let left = helper(arr.slice(start, mid))
        let right = helper(arr.slice(mid))

        let lx = left.length - 1
        let rx = right.length - 1

        while (lx >= 0 || rx >= 0) {
            if (left[lx] > right[rx]) {
                tmp.unshift(left[lx])
                ret = rx + 1 + ret
                lx --
            } else {
                tmp.unshift(right[rx])
                rx --
            }
        }
        return tmp

    }
    return ret
}
```

> 52.两个链表中的第一个公共节点

> 要使用非暴力方法外，要知道公共节点的链表一个很重要的特性：就是两个链表组成一个Y字，分别从两个链表head开始查，必将在公共节点交汇后，一直到最后。

> 思路一：遍历两个链表，各自的节点分别存入栈。遍历完，根据先进后出的原则，两个栈栈顶的元素必定相等，则继续判断。直至最后一个相等。这个算法时间复杂度为O(M+N),空间复杂度为O(M+N)

> 思路二：遍历获得两个长度，如果长度一致，那么两个一致遍历时就会遇到相同。如果A长于B,A可以先遍历A-B节点，再一起。

```javaScript
function getCommon(list1, list2) {
    let arr1 = []
    let arr2 = []

    let cur1 = list1
    let cur2 = list2
    while (cur1) {
        arr1.push(cur1.value)
        cur1 = cur1.next
    }

    while (cur2) {
        arr2.push(cur2.value)
        cur2 = cur2.next
    }
    let ret
    let po1
    let po2
    while (arr1.length || arr2.length) {
        po1 = arr1.pop()
        po2 = arr2.pop()
        if (po1 == po2) {
            ret = po1
        } else {
            break
        }
    }
    return ret
}

function getCommon(list1, list2) {
    let len1 = 0
    let len2 = 0

    let cur1 = list1
    let cur2 = list2
    while (cur1) {
        len1++
        cur1 = cur1.next
    }

    while (cur2) {
        len2++
        cur2 = cur2.next
    }
    let max = '1'
    if (len1 < len2) {
        max = '2'
    }

    let diff = Math.abs(len2 - len1)
    let tmp1 = list1
    let tmp2 = list2
    while (len1 && len2) {
        if (diff) {
            diff--
            if (max == '1') {
                tmp1 = tmp1.next
                len1 --
                continue
            } else {
                tmp2 = tmp2.next
                len2 --
                continue
            }
        }
        if (tmp1.value == tmp2.value) {
            return tmp1.value
        } else {
            tmp1 = tmp1.next
            tmp2 = tmp2.next
        }
        len2 --
        len1 --
    }
}
```

> 53.在排序数组中查找数字，就是查找某数字出现的次数。

> 排出使用遍历O(N)。那就是查找某数字出现最小的下标与最大的下标。二分法查找第一个与最后一个。

```javaScript
function getNumCount(arr, num) {
    let l = getFirstK(arr, num)
    let r = getLastK(arr, num)
    return r - l + 1
}

function getFirstK(arr, num) {
    let l = -1
    let start = 0
    let len = arr.length
    while (start < len - 1) {
        let mid = (start + len) >> 1
        let base = arr[mid]
        // 数据在左侧
        if (base < num) {
            start = mid
        // 数据在右侧
        } else if (base > num) {
            len = mid
        // 刚好相等
        } else {
            // 目标数据刚好在最左侧
            if (mid - 1 < 0 || arr[mid - 1] != num) {
                l = mid
                start = mid
                break
            } else {
                len = mid
            }
        }
    }
    return l
}

function getLastK(arr, num) {
    let r = -1
    let start = 0
    let len = arr.length
    while (start < len - 1) {
        let mid = (start + len) >> 1
        let base = arr[mid]
        // 数据在左侧
        if (base < num) {
            start = mid
        // 数据在右侧
        } else if (base > num) {
            len = mid
        // 刚好相等
        } else {
            console.log(start, mid, len, arr[mid + 1])
            // 目标数据刚好在最右侧
            if (mid + 1 >= len || arr[mid + 1] != num) {
                r = mid
                break
            } else {
                start = mid
            }
        }
    }
    return r
}
```

> 53.1 求0~n-1中缺失的数字。长度n-1, 所有数字都在0~n-1中。

> 按道理来说，索引i必须等于其值。还是二分法。

```javaScript
function getNum(arr) {
    let start = 0
    let end = arr.length - 1
    while (start < end - 1) {
        let mid = (start + end) >> 1
        let base = arr[mid]
        if (base == mid) {
            start = mid
        } else {
            end = mid
        }
    }
    return start+1
}
```

> 53.2 找出单调递增数组中元素与下标相等的值

```javaScript
function getNum(arr) {
    let start = 0
    let end = arr.length - 1
    while (start < end - 1) {
        let mid = (start + end) >> 1
        let base = arr[mid]
        if (base < mid) {
            start = mid
        } else if (base > mid) {
            end = mid
        } else {
            return base
        }
    }
    return -1
}
```

> 55. 求二叉树的最大深度

```javaScript
function getDeep(tree) {
    let ret = 0
    if (!tree) {
        return 0
    }
    let left = getDeep(tree.left)
    let right = getDeep(tree.right)
    ret = 1 + Math.max(left, right)
    return ret
}
```

> 55.1 判断是一个二叉树是否为平衡二叉树

> 可以求左右深度，然后依次递归下去，但是不是最优答案，因为从上到下，最底层会重复计算。要求每个节点只遍历一次。后续遍历，先先遍历左树，再遍历右树，最后遍历节点。所以在遍历时就存高度。

```javaScript
function check(tree) {
    return getDeep(tree) !== -1
}
function getDeep(tree) {
    if (!tree) {
        return 0
    }
    let left = getDeep(tree.left)
    if (left == -1) {
        return -1
    }
    let right = getDeep(tree.right)
    if (right == -1) {
        return -1
    }
    return Math.abs(left - right) > 1 ? -1 : 1 + Math.max(left, right)
}
```

> 57. 递增排序速度中，和为S的两个数。

> 思路：前后两个指针。

```javaScript
function getNum(arr, s) {
    let start = 0
    let end = arr.length - 1

    while (start < end) {
        if (arr[start] + arr[end] == s) {
            return [arr[start], arr[end]]
        } else if(arr[start] + arr[end] > s) {
            end--
        } else {
            start++
        }
    }
}
```

> 57.1 和为S的连续整数数列

> 指针初始为1，2.依次进行递增。

```javaScript
function getNum(s) {
    let start = 1
    let end = 2
    let ret = []
    let tmp = [start, end]
    let res = start + end
    while (start < s && end < s) {
        if (res < s) {
            end ++
            tmp.push(end)
            res += end
        } else if (res == s) {
            ret.push([...tmp])
            end ++
            tmp.push(end)
            res += end
        } else {
            let val = tmp.shift()
            res -= val
            start++
        }
    }
    return ret
}
```
