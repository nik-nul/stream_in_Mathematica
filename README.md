这两天发现 Mathematica 竟然没有 stream 这种东西，于是自己写了一个惰性的 stream ADT 用来处理无限长的列表
```mathematica
SetAttributes[stream, HoldAll]
car[stream[h_, _]] := h
cdr[stream[_, r_]] := r
emptyQ[stream[]] := True
emptyQ[stream[_, _]] := False
filter[stream[], _] := stream[]
filter[s_, pred_] := 
 If[pred@car@s, stream[car@s, filter[cdr@s, pred]], 
  filter[cdr@s, pred]]
map[stream[], _] := stream[]
map[s_, f_] := stream[f@car@s, map[cdr@s, f]]
combine[stream[], _, _] := stream[]
combine[_, stream[], _] := stream[]
combine[s1_, s2_, f_] := 
 stream[f[car@s1, car@s2], combine[cdr@s1, cdr@s2, f]]
slice[_, 0] := {}
slice[stream[], ___] := {}
slice[s_, n_] := {car@s}~Join~slice[cdr@s, n - 1]
slice[s_, 1, nd_] := slice[s, nd - 1]
slice[s_, st_, nd_] := slice[cdr@s, st - 1, nd - 1]
```
没什么好说的，我把可能能用到的函数都写了，作用都是字面意思，实现都是很直观的实现。不得不说 Pattern Matching 真香。

stream 由于是惰性的可以处理一些正常难以处理的无限长的列表，或者整点花活。
下面给几个例子
```mathematica
nat[n_] := stream[n, nat[n + 1]]
```
上面这样可以递归地实现一个无限长的从 n 开始的列表

想要偶数列表？ `filter[nat@0, #~Mod~2 == 0 &]` 

当然也可以写 `map[nat@0, 2 # &]` 

```mathematica
fact = stream[1, combine[nat@1, fact, #1*#2 &]]
```
上面这行可以写一个从 0 开始的阶乘的列表

```mathematica
sieve[s_] := 
 stream[car[s], sieve[filter[cdr[s], ! Divisible[#, car[s]] &]]]
```
上面这行是一个欧拉筛，只需要 `sieve@nat@2` 就可以得到一个无限长的素数列表

```mathematica
In[] = slice[sieve[nat@2], 12]
Out[] = {2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31, 37}
```
真优雅！
