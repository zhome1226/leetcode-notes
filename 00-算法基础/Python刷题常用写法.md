# Python 刷题常用写法

刷题时反复用到的 Python 小工具，集中记在这里。之后遇到新写法继续往下加，专题笔记里只留题意和思路，基础写法指回本页。

## 1. `range`：左闭右开

`range(n)` 得到 `0, 1, …, n-1`，**不含** `n`。  
`range(1, n)` 从 1 扫到 `n-1`。  
`range(j, len(nums))` 从下标 `j` 扫到末尾。

所以 `for i in range(len(nums))` 正好覆盖每个合法下标，不会越界。

## 2. 直接遍历元素，还是遍历下标

只关心值、不关心位置时，直接写：

```python
for c in s:
    ...
```

需要下标（要写回 `nums[i]`，或要和 `i+1` 比较）时，用 `range`：

```python
for i in range(len(nums)):
    ...
```

## 3. `enumerate`：值和下标一起拿

既要元素又要下标时，可以写成：

```python
for i, x in enumerate(nums):
    ...
```

这里 `i` 是下标，`x` 是 `nums[i]`，等价于先 `range` 再取 `nums[i]`，写起来更短。

## 4. `set`：去重和查是否出现过

`set` 里每个值只存一份，查「这个数出现过没有」很快。

```python
seen = set()
seen.add(3)
if 3 in seen:
    ...
```

注意：力扣若要求**原地改数组、保持相对顺序**（如 26、283），一般不能靠 `set` 直接交差，还是用双指针更合适。`set` 更常出现在哈希类题（两数之和、是否有重复等）。

## 5. `append` / `pop`：当栈用

把列表想成一摞盘子：

- `stack.append(x)`：往上加一个
- `stack.pop()`：拿走最上面一个（空列表上 `pop` 会报错）

844、括号匹配这类题，常用列表当栈。

## 6. `if stack:`：判非空，不是判 `None`

空列表 `[]` 在 `if` 里算假，但它**不是** `None`。  
`None` 表示「根本没有这个对象」；`[]` 表示「有一个空列表」。

所以「栈里还有东西才 pop」写成：

```python
elif stack:
    stack.pop()
```

不要写成 `if stack != None`——空栈也不是 `None`，那样空栈仍会去 `pop` 然后报错。

## 7. `elif` 和 `else`

- `elif 条件:`：前面的 `if` 没进，再判断这个条件
- `else:`：前面都没进，直接兜底，不再加条件

`elif stack: stack.pop()` 等价于 `else: if stack: stack.pop()`，只是更短。

## 8. 字符串不能按下标改

Python 字符串是不可变的，`s[i] = 'a'` 会报错。要「改字符」时，通常：

- 用列表当栈 / 缓冲区，最后 `''.join(stack)`，或
- 拼出新字符串

## 9. 空列表不能按下标写

`seen = []` 再写 `seen[0] = x` 会越界。要么先开满长度：

```python
seen = [0] * len(nums)
```

要么用 `append` 往后面加。

## 10. 原地修改 vs 重新赋值

`nums = seen` 只是让函数里的名字指向新对象，**改不了**外面传进来的那份数组。

要改原列表内容，用：

```python
nums[i] = ...
# 或整段替换
nums[:] = seen
```

题目写 `modify in-place` / `rtype: None` 时，尤其要注意这一点。

## 11. 内嵌函数不要写 `self`

类方法第一个参数是 `self`；写在方法里面的小函数是普通函数，参数列表里**不要**抄 `self`：

```python
def build(char):
    ...
```

## 12. 二分的 `mid` 写法

算中点时习惯写成：

```python
mid = left + (right - left) // 2
```

和 `(left + right) // 2` 在 Python 里结果一样，但养成这个写法，以后换到可能溢出的语言也不容易踩坑。

## 13. `float('inf')`：用正无穷初始化最小值

```python
min_len = float('inf')
```

`float('inf')` 表示正无穷大，比任何有限数字都大。要求最小值时，先设成它，第一次遇到实际候选值就能更新：

```python
min_len = min(min_len, 4)  # 从正无穷变成 4
min_len = min(min_len, 2)  # 更新为 2
min_len = min(min_len, 3)  # 仍然是 2
```

第 209 题中，如果结束后 `min_len` 仍是正无穷，说明从未找到达标窗口，应返回 `0`。不能初始化为 `0`，否则 `min(0, 正整数长度)` 永远是 `0`。

## 14. `for` 和 `while`：看循环条件与指针如何移动

`for` 从可迭代对象中依次取值；`while` 每轮执行前检查条件，条件为真就继续，循环涉及的指针通常需要自己更新。例如：

```python
for right in range(len(nums)):
    print(nums[right])
```

也可以写成：

```python
right = 0
while right < len(nums):
    print(nums[right])
    right += 1
```

对这里的 `range`，下一轮的 `right` 由迭代器提供，因此 `for` 内不必手动 `right += 1`；`while` 若忘记推进指针，可能一直重复执行。

### 第 209 题：两个 `for` 和两个 `while` 的区别

朴素枚举可以用两个 `for`，每换一个起点，就重新向右累加：

```python
# 放在已初始化 min_len 的求解函数中
for left in range(len(nums)):
    cur_sum = 0
    for right in range(left, len(nums)):
        cur_sum += nums[right]
        if cur_sum >= target:
            min_len = min(min_len, right - left + 1)
            break  # 对这个起点，继续向右只会更长
```

这种写法会反复计算重叠区间。即使有 `break`，最坏情况下（例如目标大于整个数组总和）仍要做 `n + (n-1) + ... + 1` 次累加，时间复杂度为 `O(n²)`。

第 209 题的滑动窗口用两个 `while`，但 `left` 不会在外层每轮重新归零，`cur_sum` 也会沿用。左右边界在整个过程中各最多向右移动 `n` 次，所以时间复杂度是 `O(n)`。

外层也可以改成 `for`，保留内层 `while`，算法和复杂度不变：

```python
# 放在求解函数中，target 是目标值
left = 0
cur_sum = 0
min_len = float('inf')

for right in range(len(nums)):
    cur_sum += nums[right]
    while cur_sum >= target:
        min_len = min(min_len, right - left + 1)
        cur_sum -= nums[left]
        left += 1
```

外层负责依次加入右端元素，内层负责在达标时反复移出左端元素。不要把内层机械改成每轮重新枚举的 `for left in range(...)`，否则会破坏窗口状态的维护。

**复杂度取决于总执行次数与指针是否重置，不能只看用了 `for` 还是 `while`。** 两个 `for` 不一定是 `O(n²)`，两个 `while` 也不一定是 `O(n)`。

## 15. `return A if 条件 else B`：条件表达式

```python
return min_len if min_len != float('inf') else 0
```

结构是 `A if 条件 else B`：先判断条件，成立就选择 `A`，否则选择 `B`，然后 `return` 把所选结果返回。

上面的写法等价于：

```python
if min_len != float('inf'):
    return min_len
else:
    return 0
```

这里 `!=` 表示“不等于”：找到过有效长度就返回该长度；仍是正无穷说明没找到，返回 `0`。条件表达式只会求值被选中的分支，不会把两边都执行一遍。

## 以后怎么维护

每学会一个会反复用的写法，用两三句话加一小节即可。专题笔记里若又讲到同一点，可以只写「见 [Python 刷题常用写法](../00-算法基础/Python刷题常用写法.md)」。
