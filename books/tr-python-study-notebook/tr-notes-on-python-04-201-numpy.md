---
title: "NumPy"
---

```python
np.nonzero(is_prime_arr[2:])[0] + 2
```
`is_prime_err[2:]`: 配列から先頭2要素をスキップ
`np.nonzero()`: 配列内で`True`または非ゼロ要素があるインデックスを取得
`[0]`: `np.nonzero()`の結果はタプルとして返却される。1次元配列の場合、そのタプルの最初の要素(インデックスの配列)を取得している。
`+ 2`: インデックスを元の配列に対応させるため、スキップした2を全てのインデックスに追加

NumPy配列のスライスは(pure Pythonと異なり)配列のビューを生成する。
```python
np_arr = np.ones(10, dtype=np.bool)
print(np_arr)
# [ True  True  True  True  True  True  True  True  True  True]
np_subarr = np_arr[:2]
np_subarr[:] = False
print(np_arr)
# [False False  True  True  True  True  True  True  True  True]
```
