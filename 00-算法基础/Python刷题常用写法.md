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

## 以后怎么维护

每学会一个会反复用的写法，用两三句话加一小节即可。专题笔记里若又讲到同一点，可以只写「见 [Python 刷题常用写法](../00-算法基础/Python刷题常用写法.md)」。
