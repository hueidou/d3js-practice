d3js入门之数据绑定


引子
d3js 是一款上手容易的js类库,专门用于绘制svg图形图表,其关键理念为data-join 意即数据绑定.搞清这个概念非常重要,它将以简洁优雅的形式体现数据驱动编程.
以下是Thinking with Joins的拙译 ,原作者Mike Bostock
假设你要用D3画一副散点图,因此需要生成一些 SVG circle 元素来直观地展现数据. 你会惊讶地发现D3没有提供原生的产生多个DOM元素的接口,
是的,只有一个 append 方法,用于产生单个DOM元素:
svg.append("circle")
    .attr("cx", d.x)
    .attr("cy", d.y)
    .attr("r", 2.5);

但那只是单个圆,而你想要更多: 最好data*中每个元素对应一个圆. 在你用蛮力写循环把圆画出来之前,让我们看看D3中的一个例子:
svg.selectAll("circle")
    .data(data)
  .enter().append("circle")
    .attr("cx", function(d) { return d.x; })
    .attr("cy", function(d) { return d.y; })
    .attr("r", 2.5);

*此处 data是一个 JSON 数组,其每个元素 由 x 和 y属性构成, 例如: [{"x": 1.0, "y":1.1},{"x": 2.0, "y":2.5}, …]. 另,SVG circle元素用cx,cy表达圆心坐标,r表达半径长度.
这份代码符合你的需求,即每个元素产生一个圆 , 通过x和y属性表达圆心的坐标.


但selectAll("circle")是什么意思,为什么要在产生所有圆之前去选中根本不存在的元素呢?


原来事情是这样的:告诉D3你的目标,而不要告诉它具体怎么做. 在这个例子中,D3知道我们的意图是,要让选中的"circle"元素来响应数据的变化, selectAll即描述了这个目标；而无需一步步指挥D3产生多个圆.这个概念即data-join.


data-join的背后执行了以下步骤:
selectAll("circle") 返回了一个空的选择
空的选择通过 data()方法将数据和DOM元素绑定,并产生三个虚拟的子集: enter, update and exit. enter()方法包含了待添加的数据及相应的DOM元素的占位符；update()包含了已与数据绑定的现有元素.剩下待移除的部分被包含在 exit ()方法中
一开始选择的结果是空的,因此所有数据都是待添加,将全部出现在enter的结果中.
无需循环,通过.enter().append("circle")将待添加的元素一次性加入到SVG容器.

为什么要这么麻烦呢? 为什么不直接提供原生接口? data-join的优雅之处在于抽象和解耦.上述代码在enter()里只是专心处理新增的元素,而update and exit分别专注于处理更新和待删除部分.这意味着你不用把所有DOM元素删了重绘,因此得以轻松应对实时变化的数据,甚至支持一些交互(如拖动)与渐变的效果!
这里是一个处理三种状态(增改删)的例子:
var circle = svg.selectAll("circle")
    .data(data);

circle.enter().append("circle")
    .attr("r", 2.5);

circle
    .attr("cx", function(d) { return d.x; })
    .attr("cy", function(d) { return d.y; });

circle.exit().remove();

如果我们重复运行代码,它会每次重新计算 data-join. 如果新的数据集比原来的少,多余元素会出现在 exit 中并被remove()删除.反之亦然,新增的数据出现在enter()中通过append()添加DOM元素.若新老数据集大小不变,则所有数据只是更新坐标.(译注:上文中介于enter和exit之间的代码,update()会被隐式调用)
以joins的方式思考同时让你的代码更直观: 处理这三种状态的代码无需条件(if)和循环(for)分支,只需简单描述让图形去响应数据的变化即可.如果给定的enter, update 或 exit 的选择结果为空,则会自动跳过相应的代码块,以降低性能开销.
Joins 支持在特定状态(增/删/改)下执行操作.例如,可以在enter而非update代码块中,指定静态的attributes(例如圆的半径,用 "r" attribute指定) . 仰赖于精确改动目标元素和最小化DOM变更,你已经极大地提升了浏览器渲染的表现! 类似地,你可以在特定状态下表现渐变等动画效果. 例如新增的圆可以从无到有渐变(半径从0到2.5):
circle.enter().append("circle")
    .attr("r", 0)
  .transition()
    .attr("r", 2.5);

待删除的圆也可以逐渐收缩直至消失:
circle.exit().transition()
    .attr("r", 0)
    .remove();

相信现在你已经学会用joins的方式思考了!