# **slice()、splice()、split()**

## **slice()**

> The **slice()** method returns a ***shallow copy\*** of a portion of an array into a new array object selected from ***begin to end (end not included)\***. The original array will not be modified.

複製開始與結束點（結束點不算）中的內容

對象：
*可操控Array及String*

```javascript
arr.slice()
arr.slice(begin)
arr.slice(begin, end)
```

用法：
begin 為開始的索引值，負數代表從後面開始算起，-1為倒數第一個元素。
end 為結束的索引值，沒有填寫時則得到arr中的所有元素。

因為是做shallow copy，所以原值不改變。



```javascript
var fruits = ['Banana', 'Orange', 'Lemon', 'Apple', 'Mango'];
var fruit1 = fruits.slice(1);
var fruit2 = fruits.slice(1, 3);
var fruit3 = fruits.slice(-3);

// fruits contains ['Banana', 'Orange', 'Lemon', 'Apple', 'Mango']
// fruit1 contains ['Orange', 'Lemon', 'Apple', 'Mango']
// fruit2 contains ['Orange', 'Lemon']
// fruit3 contains ["Lemon", "Apple", "Mango"]
```



## **splice()**

> The **splice()** method changes the contents of an array by removing existing elements and/or adding new elements.

從Array中添加/刪除項目，回傳被刪除的項目。

對象：
*可操控Array*

```
array.splice(start)
array.splice(start, deleteCount)
array.splice(start, deleteCount, item1, item2, ...)
```

用法：
start 增加/刪除項目的位置，負數代表從後方算起。
deleteCount 刪除的個數，如為0則不會刪除。
item… 添加的新項目。

會改變原值

```javascript
var myFish1 = ['angel', 'clown', 'drum', 'sturgeon'];
var removed1 = myFish1.splice(2, 1, 'trumpet');

// myFish1 is ["angel", "clown", "trumpet", "sturgeon"]
// removed1 is ["drum"]

var myFish2 = ['angel', 'clown', 'drum', 'sturgeon'];
var removed2 = myFish2.splice(-2, 2, 'trumpet');

// myFish2 is ["angel", "clown", "trumpet"]
// removed2 is ["drum", "sturgeon"]
```



## split()

> The **split()** method splits a String object into an array of strings by separating the string into substrings.

分割字串成字串組

對象：
*可操控String*

```
stringObject.split(separator,howmany)
```

用法：
separator 字串符或正則表達式，從該參數指定的地方分割stringObject。
howmany 返回值的最大長度，超過該長度則不顯示。

不改變原值



```javascript
var str="How are you ?"
var splits1 = str.split(" ");
var splits2 = str.split("");
var splits3 = str.split(" ",3);

//splits1 contains ["How", "are", "you", "?"]
//splits2 contains ["H", "o", "w", " ", "a", "r", "e", " ", "y", "o", "u", " ", "?"]
//splits3 contains ["How", "are", "you"]
```

