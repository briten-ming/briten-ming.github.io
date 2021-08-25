# 多个不等宽按钮自适应
考虑如下场景：
一个编辑器组件，上方是一个 Toolbar，其中有若干宽度不定的操作按钮，下方是内容区域。

现在面临一个需求：我希望 Toolbar 总是只占一行的高度，因为假如按钮数量不固定，按钮过多导致的换行将会挤压内容区域。  
业务上的解决方案是 Toolbar 默认只展示一行，多行情况通过一个「更多」的功能来实现 Toolbar 面板展开与收起。
  
### 实现思路
将 Toolbar 分成两列，一列放所有的操作按钮，自适应宽度且 `overflow: hidden` ，另一列固定宽度放「更多」按钮，每次点击「更多」都会让操作按钮列表的高度在 `auto` 和 `固定高度` 之间切换。
![image](https://user-images.githubusercontent.com/28632663/130792402-30652294-5d0f-4f4e-b310-239b2b19526b.png)

### 问题
功能实现了，但是有一个潜在问题，如果容器宽度足够在一行内放下所有操作按钮，那么「更多」按钮就没有实际的用处了，此时我们希望能不要显示「更多」 
这个问题我也尝试了很多方法，最后使用 `ResizeObserver` 实现了。

### 实现
- 操作按钮列表高度默认给 `auto` 
- 组件挂载时用 `ResizeObserver` 监听并获得组件高度
- 若高度超出一行，则修改操作按钮列表高度为一行
- 关闭监听

```js
  const resizeObserver = useMemo(() => {
    return new ResizeObserver((entries) => {
      const { height } = entries?.[0]?.target?.getBoundingClientRect();
      if (height <= 0) return;
      const count = Math.ceil(height / 40);
      setShowMore(count > 1);
      resizeObserver.disconnect();
    });
  }, []);

  useLayoutEffect(() => {
    resizeObserver.observe(btnListRef.current);
    return () => resizeObserver.disconnect();
  }, [resizeObserver]);
```

在线 [Demo](https://codesandbox.io/s/resizeobserver-nv8wp?file=/src/App.js)
