# regular expression matching

---

```javascript
var isMatch = function(s, p) {
    if (s === p) {
        return true
    }
    if (p.length === 0) {
        if (s.length === 0) {
            return true
        } else {
            return false
        }
    }
    if (p.length === 1) {
        if (s === p || p === '.') {
            return true
        } else {
            return false
        }
    }
    if (p[1] !== '*') {
        if (!s.length) {
            return false
        }
        if (s[0] === p[0] || p[0] === '.') {
            return isMatch(s.substr(1), p.substr(1))
        } else {
            return false
        }
    }
    while (s.length && (s[0] === p[0] || p[0] === '.')) {
        if (isMatch(s, p.substr(2))) {
            return true
        }
        s = s.substr(1)
    }
    return isMatch(s, p.substr(2))
};
```