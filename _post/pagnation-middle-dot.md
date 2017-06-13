# 分页算法

### 实现目的：

```basic
> getPage(5,3,2);
[ 1, 2, 3, 4, 5 ]
> getPage(6,3,2);
[ 1, 2, 3, 4, 5, 6 ]
> getPage(7,3,2);
[ 1, 2, 3, 4, 5, '...', 7 ]
> getPage(10,3,2);
[ 1, 2, 3, 4, 5, '...', 10 ]
> getPage(10,4,2);
[ 1, 2, 3, 4, 5, 6, '...', 10 ]
> getPage(10,5,2);
[ 1, '...', 3, 4, 5, 6, 7, '...', 10 ]
> getPage(10,6,2);
[ 1, '...', 4, 5, 6, 7, 8, '...', 10 ]
> getPage(10,8,2);
[ 1, '...', 6, 7, 8, 9, 10 ]
> getPage(10,9,2);
[ 1, '...', 6, 7, 8, 9, 10 ]
> getPage(10,10,2);
[ 1, '...', 6, 7, 8, 9, 10 ]
```



### 实现方法1（推荐）：

```javascript
 /**
  * [getPage 分页算法]
  * @param  {[number]} count [总页数]
  * @param  {[number]} cur   [当前页]
  * @param  {[number]} w     [当前页的两侧页码数]
  * @return {[array]} res      [返回分页形式数组]
  * author: weij@chsi.com.cn
  */
function getPage(count, cur, w){
  if(count<2){return;} // 只有一页无需显示
  var pageStart = 1; //初定乾坤，起始于 1
  var pageEnd = count; //初定乾坤，终结于 count

  var pageMiddle=[]; // 中间页码的显示变化莫测

  var res =[];
  var isneedStart=true;
  var isneedEnd = true;

  w = w || 2;  // 当前页码左右默认2个页码
  var p1 = cur-w > 2 ? true : false; // 前缀判定出现...
  var p2 = cur+w < count -1 ? true : false; // 后缀判断出现...

  for(var i=0;i<(w*2+1);i++){
    if(pageStart+i==pageEnd){ isneedEnd = false;}
    if(pageStart+i>pageEnd){ break;} // 遇到末页停止循环
    if(pageStart+1>cur-w){ // 处理前部分
      isneedStart = false;
      pageMiddle.push(pageStart+i);
      continue;
    }
    if(cur+w+1>pageEnd){ // 处理后部分
      isneedEnd = false;
      pageMiddle.push(pageEnd-w*2+i);
       continue;
    }
    pageMiddle.push(cur-w+i);
  }


if(isneedStart){ res.push(pageStart);}
if(p1) { res.push('...');}
  res = res.concat(pageMiddle);
if(p2) { res.push('...');}
if(isneedEnd){ res.push(pageEnd);}


return res;
}
getPage(10,3,2);

// > getPage(10,3,2);
// [ 1, 2, 3, 4, 5, '...', 10 ]
// > getPage(10,4,2);
// [ 1, 2, 3, 4, 5, 6, '...', 10 ]
// > getPage(10,5,2);
// [ 1, '...', 3, 4, 5, 6, 7, '...', 10 ]
// > getPage(10,6,2);
// [ 1, '...', 4, 5, 6, 7, 8, '...', 10 ]
// > getPage(10,8,2);
// [ 1, '...', 6, 7, 8, 9, 10 ]
// > getPage(10,9,2);
// [ 1, '...', 6, 7, 8, 9, 10 ]
// > getPage(10,10,2);
// [ 1, '...', 6, 7, 8, 9, 10 ]
```



### 实现方法2：

```javascript
**
 * [getPage2 果果哥给出的分页算法
 * @param  {[type]} count [description]
 * @param  {[type]} cur   [description]
 * @param  {[type]} w     [description]
 * @return {[type]}       [description]
 * author: 果果
 */
function getPage2(count, cur, w){

  var ret = [];
  w = Math.max(0, w) || 2;
  cur = Math.min(count, cur);

  //计算中间部分
  var start = Math.max(0, cur - w - 1);
  var end = Math.min(cur + w, count);
    for (var i = start; i < end; i++) {
    ret.push(i + 1);
    }

  //补全头尾
  if (ret[0] > 1) {
    ret.unshift(1);
  }
  if (ret[ret.length -1] < count) {
    ret.push(count);
  }

  //补全头尾与中间的过渡部分
  var diff = ret[1] - ret[0];
  if (diff == 2) {
    ret.splice(1,0,2);
  }else if(diff > 2){
    ret.splice(1,0, '...');
  }

  var last = ret.length -1;
  diff = ret[last] - ret[last-1];
  if (diff == 2) {
    ret.splice(last,0, count - 1);
  }else if(diff > 2){
    ret.splice(last,0, '...');
  }

  return ret;
}

getPage2(10,1,4);

// > getPage2(10,5,4);
// [ 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 ]
// > getPage2(10,5,2);
// [ 1, 2, 3, 4, 5, 6, 7, '...', 10 ]
// > getPage2(10,2,2);
// [ 1, 2, 3, 4, '...', 10 ]
// > getPage2(10,1,2);
// [ 1, 2, 3, '...', 10 ]
// > getPage2(10,2,2);
// [ 1, 2, 3, 4, '...', 10 ]
// > getPage2(10,3,2);
// [ 1, 2, 3, 4, 5, '...', 10 ]
// > getPage2(10,4,2);
// [ 1, 2, 3, 4, 5, 6, '...', 10 ]
// > getPage2(10,5,2);
// [ 1, 2, 3, 4, 5, 6, 7, '...', 10 ]
// > getPage2(10,6,2);
// [ 1, '...', 4, 5, 6, 7, 8, 9, 10 ]
```

