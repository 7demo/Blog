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
