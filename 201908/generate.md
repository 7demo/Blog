# Pascal's Triangle

> 动态规划，每一行的元素都是决定下一行元素。复杂度为O(n^2)。注意每行循环的时候可以跳过开始一个与最后一个。

```javascript
var generate = function(numRows) {
    let arr = Array.from({
        length: numRows
    }).map(item=> {
        return [1]
    })
    if (numRows > 1) {
        arr[1] = [1,1]
    }
    for (let i = 2; i < numRows; i++) {
        for (let j = 1; j < i; j++) {
            arr[i][j] = arr[i - 1][j - 1] + arr[i - 1][j]
        }
        arr[i].push(1)
    }
    return arr
};
```