---
title: 在 React 项目中使用 dagre-d3
date: 2016-06-24T09:20:02.322Z
---

React 已经成为了公司所有项目的前端必选框架了, JavaScript 社区非常活跃，而 React 也几乎成为了前端最火热的框架，所以各种知名的库几乎都已经有了 React 的版本，比如 d3, 也有了了 [react-d3](https://github.com/esbullington/react-d3), 但是很可惜的是，还没有完全移植完成，还只能支持基本的几种形式，当然对于基础的 Web 应用也够用。可以偏偏有向无回路图 (Directed Acylic Graph) 还没有，那怎么办呢，同事说 [airflow](https://github.com/apache/incubator-airflow) 使用了 [dagre-d3](https://github.com/cpettitt/dagre-d3)，看了一下，使用起来很简单，效果也还不错，要不试试在 React 中试试吧。

其实要在 React 项目中使用 dagre-d3 只需要这几步就够了.


* 先绘制几个点

```
  const dagGraph = new dagreD3.graphlib.Graph().setGraph({});
  ['A', 'B', 'C'].forEach((name) => {
    dagGraph.setNode(name, { label: name });
  });
  dagGraph.setEdge('A', 'B', {});
```


* 获取你的 DOM node

```
  const svgDOMNode = ReactDOM.findDOMNode(this.refs.svg);
  const svg = d3.select(svgDOMNode);
  const inner = svg.append('g');
```

我们的图当然要绘制在 <svg> 上，但是为什么需要在 <g> 里面呢，后面你就知道奇妙之处了。

* render 到你的 DOM 上吧

```
  const render = new dagreD3.render();
  render(inner, dagGraph);
```

由于有获取你真正的 DOM 节点，所以你需要将上面代码的执行放到 componentDidMount之后, 并且你应该这样。

```
 componentDidMount() {
  setTimeout(() => {
    // your above codes
  }, 0);
 }

```

是不是看到你优美的 DAG 图了呀，可以怎么不能缩放呢，好吧，然我们来给我的 DAG 加上 zoom 支持吧。

* zoom 支持

```
  const zoom = d3.behavior.zoom().on('zoom', () => {
    this.autoLayoutDag();

    const translateFunc = `translate(${d3.event.translate})`;
    const scaleFunc = `scale(${d3.event.scale})`;
    inner.attr('transform', `${translateFunc} ${scaleFunc}`);
  });
  svg.call(zoom);

  const render = new dagreD3.render();
  render(inner, graph);

  const initialScale = 0.80;
  const transXY = [(svg.attr('width') - graph.graph().width * initialScale) / 2, 20];
  zoom.translate(transXY).scale(initialScale).event(svg);
```

这就是如何在 React 项目中使用 dagre-d3 来绘制有向图。当然，其实其他还没有 React-ready 的库也同样可以自己摸索一下，基本上都可以很愉快在 React 中使用哦。

