### 常见排序算法

```python
def bubble_sort(lst):
    for i in range(len(lst) - 1, 0, -1):
        exchange = False
        for j in range(i):
            if lst[j] > lst[j + 1]:
                exchange = True
                lst[j], lst[j + 1] = lst[j + 1], lst[j]
        if not exchange:
            break


a_list = [54, 26, 93, 17, 77, 31, 44, 55, 20]
bubble_sort(a_list)
print("bubble sort", a_list)


def select_sort(lst):
    for i in range(len(lst) - 1, 0, -1):
        max_index = 0
        for j in range(1, i + 1):
            if lst[j] > lst[max_index]:
                max_index = j
        lst[i], lst[max_index] = lst[max_index], lst[i]

# 另外想的解法
# def select_sort(lst):
#     for i in range(len(lst) - 1, 0, -1):
#         max_index = i
#         for j in range(i):
#             if lst[j] > lst[max_index]:
#                 max_index = j
#         lst[i], lst[max_index] = lst[max_index], lst[i]

a_list = [54, 26, 93, 17, 77, 31, 44, 55, 20]
select_sort(a_list)
print("select sort", a_list)


def insert_sort(lst):
    for i in range(1, len(lst)):
        for j in range(i, 0, -1):
            if lst[j - 1] > lst[j]:
                lst[j - 1], lst[j] = lst[j], lst[j - 1]
            else:
                break

# 另外想的解法
# def insert_sort(lst):
#     for i in range(len(lst)-1):
#         for j in range(i + 1, len(lst)):
#             while j > 0 and lst[j] < lst[j - 1]:
#                 lst[j], lst[j-1] = lst[j-1], lst[j]
#                 j -= 1

a_list = [54, 26, 93, 17, 77, 31, 44, 55, 20]
insert_sort(a_list)
print("insert sort", a_list)


def shell_sort(lst):
    n = len(lst)
    gap = n // 2
    while gap > 0:
        for i in range(gap, n):
            j = i
            while j >= gap and lst[j - gap] > lst[j]:
                lst[j - gap], lst[j] = lst[j], lst[j - gap]
                j -= gap
        gap = gap // 2


a_list = [54, 26, 93, 17, 77, 31, 44, 55, 20]
shell_sort(a_list)
print("shell sort", a_list)


def quick_sort(lst, start, end):
    if start >= end:
        return

    mid = lst[start]
    low = start
    high = end

    while low < high:
        while low < high and lst[high] >= mid:
            high -= 1
        lst[low] = lst[high]

        while low < high and lst[low] <= mid:
            low += 1
        lst[high] = lst[low]

    lst[low] = mid
    quick_sort(lst, start, low - 1)
    quick_sort(lst, low + 1, end)


a_list = [54, 26, 93, 17, 77, 31, 44, 55, 20]
quick_sort(a_list, 0, len(a_list) - 1)
print("quick sort", a_list)


def merge_sort(lst):
    if len(lst) <= 1:
        return lst
    mid = len(lst) // 2
    left = merge_sort(lst[:mid])
    right = merge_sort(lst[mid:])

    merged = []
    while left and right:
        if left[0] < right[0]:
            merged.append(left.pop(0))
        else:
            merged.append(right.pop(0))
    merged.extend(left if left else right)
    return merged


a_list = [54, 26, 93, 17, 77, 31, 44, 55, 20]
print("merge sort", merge_sort(a_list))

```



### 字符问题

报错： UnicodeDecodeError: 'utf8' codec can't decode byte 0x82 in position 35

然后google搜索 `json dumps error ignored`

```python
json.dumps(packet, default=lambda o: '<not serializable>')
```

https://stackoverflow.com/questions/51674222/how-to-make-json-dumps-in-python-ignore-a-non-serializable-field