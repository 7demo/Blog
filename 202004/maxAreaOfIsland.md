# 岛屿的数量

> 回溯法，遇到岛屿是四个方向相加

```javaScript
var maxAreaOfIsland = function(grid) {
    if (!grid || !grid.length) {
        return 0
    }
    let yLen = grid.length
    let xLen = yLen ? grid[0].length : 0
    let visited = []
    for (let i = 0; i < yLen; i++) {
        visited[i] = []
    }
    let ret = 0
    for (let i = 0; i < xLen; i++) {
        for (let j = 0; j < yLen; j++) {
            let tmp = help(i, j, xLen, yLen, visited)
            ret = tmp > ret ? tmp : ret
        }
    }
    function help(i, j, xLen, yLen, visited) {
        if (i < 0 || j < 0 || i >= xLen || j >= yLen || visited[j][i]) {
            return 0
        }
        visited[j][i] = true
        if (grid[j][i] == '1') {
            let tmp1 = help(i+1, j, xLen, yLen, visited)
            let tmp2 = help(i-1, j, xLen, yLen, visited)
            let tmp3 = help(i, j-1, xLen, yLen, visited)
            let tmp4 = help(i, j+1, xLen, yLen, visited)
            return tmp1 + tmp2 + tmp3 + tmp4 + 1
        } else {
            return 0
        }
    }
    return ret
};
```
