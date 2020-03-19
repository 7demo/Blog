# 盛最多水的容器

```javaScript
var maxArea = function(height) {
    let max = 0
    let start = 0
    let end = height.length - 1
    while (start < end) {
        let width = end - start
        let l = height[start]
        let r = height[end]
        let area = 0
        if (l < r) {
            area = l * width
            start++
        } else {
            area = r * width
            end--
        }
        max = Math.max(max, area)
    }
    return max
};
```
