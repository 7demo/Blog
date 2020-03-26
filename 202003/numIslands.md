# numIslands

> 就是回溯法

```javaScript
function checkIslands(arr, i, j, iend, jend, isCheck, flag) {
    if (i < 0 || j < 0 || i > iend || j > jend || isCheck[j][i]) {
        return flag
    }
    isCheck[j][i] = true
    if (arr[j][i] == '0') {
        flag = flag || false
    } else {
        let x1flag = checkIslands(arr, i + 1, j, iend, jend, isCheck, true)
        let x2flag = checkIslands(arr, i - 1, j, iend, jend, isCheck, true)
        let y1flag = checkIslands(arr, i, j + 1, iend, jend, isCheck, true)
        let y2flag = checkIslands(arr, i, j - 1, iend, jend, isCheck, true)
        flag = x1flag || x2flag || y1flag || y2flag
    }
    return flag
}
var numIslands = function(grid) {
    let ret = 0
    if (!grid || !grid.length) {
        return []
    }
    let yEnd = grid.length - 1
    let xEnd = grid[0].length - 1
    let isCheck = []
    for (let i = 0; i <= yEnd; i++) {
        isCheck[i] = []
    }
    for (let y = 0; y <= yEnd; y++ ) {
        for (let x = 0; x <= xEnd; x++) {
            let flag = checkIslands(grid, x, y, xEnd, yEnd, isCheck, false)
            if (flag) {
                ret ++
            }
        }
    }
    return ret
}
```
