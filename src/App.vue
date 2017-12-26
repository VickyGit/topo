<template>
  <div id="app" class="container">
    <!--<div id="topo" class="topo" ref="topo"></div>-->
    <button type="button" class="exit" @click="jsBack">退出</button>
  </div>
</template>

<script type="text/ecmascript-6">
  import * as d3 from 'd3'

  export default {
    name: 'app',
    data () {
      return {
        relation: {},
        cname: ''
      }
    },
    methods: {
      jsBack () {
//        返回按钮
        var u = navigator.userAgent
        var isAndroid = u.indexOf('Android') > -1 || u.indexOf('Linux') > -1
        var isiOS = !!u.match(/\(i[^;]+;( U;)? CPU.+Mac OS X/) //ios终端
        console.log('android test')
        if (isAndroid) {
//          Android端
          console.log('is android')
          window.android.back()
        } else if (isiOS) {
//          ios端
        } else {
//          pc端
        }
      },
      showd3 () {
//        获取body高度和宽度
        let height = document.body.clientHeight
        let width = document.body.clientWidth

        //移动端设备横竖屏重新加载页面
        function change () {
          window.location.reload()
        }

        window.addEventListener('onorientationchange' in window ? 'orientationchange' : 'resize', change, false)
//        节点大小（圆圈大小）
        const nodeSize = 35
//        初始化时连接线的距离长度
        const linkDistance = 130
//        赋值数据集
        var nodes = this.relation.nodes
        var links = this.relation.links
        console.log(nodes)

//      设置画布，获取id为app的对象，添加svg，这里的图像用了svg，意为可缩放矢量图形，它与其他图片格式相比较，svg更加小，因为是矢量图，放大不会失帧。具体可以自行百度svg相关知识
        var svg = d3.select('#app').append('svg')
          .attr('xmlns', 'http://www.w3.org/2000/svg')
          .attr('version', '2.0')
          .attr('class', 'svg')//给svg设置了一个class样式，主要作用是长宽设置为100%
//        设置力布局，使用d3 v4版本的力导向布局
        var force = d3.forceSimulation()
          .force('center', d3.forceCenter(width / 2, height / 2))//设置力导向布局的中心点，创建一个力中心，设置为画布长宽的一半，所以拓扑图会在画布的中心点
          .force('charce', d3.forceManyBody().strength(-60))//节点间的作用力，如果不设置.strength(-60）的话，默认是-30
          .force('collide', d3.forceCollide())//使用默认的半径创建一个碰撞作用力。radius默认所有的节点都为1

//        设置缩放
//        svg下嵌套g标签，缩放都在g标签上进行
        var g = svg.append('g')
//        d3.zoom是设置缩放，pc端是滚轮进行缩放，在移动端可以通过两指进行缩放
        var zoomObj = d3.zoom()
          .scaleExtent([0.5, 1.2]) // 设置缩放范围
          .on('zoom', () => {
            //监听zoom事件，zoom发生时，调用该方法
            const transform = d3.event.transform //获取缩放和偏移的数据，不懂得同学可以自行通过console.log(d3.event.transform)滑动滚轮查看数据变化
            g.attr('transform', transform)   // 设置缩放和偏移量 transform对象自带toString()方法
          })
          .on('end', () => {
//            该方法在缩放时间结束后回调
            // code
          })
        svg.call(zoomObj)
//        绘制箭头
        //箭头
        // eslint-disable-next-line no-unused-vars
        var marker =
          g.append('marker')
            .attr('id', 'resolved')
            //.attr("markerUnits","strokeWidth")//设置为strokeWidth箭头会随着线的粗细发生变化
            .attr('markerUnits', 'userSpaceOnUse')//用于确定marker是否进行缩放。取值strokeWidth和userSpaceOnUse，
            .attr('viewBox', '0 -5 10 10')//坐标系的区域
            .attr('refX', 39)//箭头坐标
            .attr('refY', 0)
            .attr('markerWidth', 12)//标识的大小
            .attr('markerHeight', 12)
            .attr('orient', 'auto')//绘制方向，可设定为：auto（自动确认方向）和 角度值
            .attr('stroke-width', 2)//箭头宽度
            .append('path')
            .attr('d', 'M0,-5L10,0L0,5')//箭头的路径
            .attr('fill', '#ff7438')//箭头颜色

//        设置连线
        var edgesLine = g.selectAll('line')
          .data(links)
          .enter()
          .append('path')
          .attr('class', 'edgelabel')//添加class样式
          .style('stroke', '#ff7438')//添加颜色
          .style('stroke-width', 1)//连接线粗细度
          .attr('marker-end', 'url(#resolved)')//设置线的末尾为刚刚的箭头
//        设置连接线中间关系文本
        var edgesText = g.selectAll('.linetext')
          .data(links)
          .enter()
          .append('text')
          .attr('class', 'linetext')
          .text((d) => {
//          设置关系文本
            return d.relation
          })
//        设置拖拽
        var drag = d3.drag()
          .on('start', (d, i) => {
            if (!d3.event.active) {
//              拖拽开始回调
              force.alphaTarget(0.1).restart() // 这个方法可以用在在交互时重新启动仿真，比如拖拽了某个节点，重新进行布局。这个必须要进行设置不然会拖动不了。
            }
            d.fixed = true //偏移后固定不动
//            d3.event.sourceEvent.stopPropagation()
            d.fx = d.x//记录当前默认位置（x - 节点当前的 x-位置，如果要为某个节点设置默认的位置，则需要为该节点设置如下两个属性:fx =x位置）
            d.fy = d.y
          })
          .on('drag', (d, i) => {
//            拖动时，设置拖动后默认位置的x，y
            d.fx = d3.event.x
            d.fy = d3.event.y
          })
          .on('end', (d, i) => {
//            拖动结束后
            if (!d3.event.active) {
              force.alphaTarget(0)
            }
          })

//        设置节点
        var node = g.selectAll('circle')
          .data(nodes)
          .enter()
          .append('circle')
          .attr('r', nodeSize)
          .attr('class', (d, i) => {
//            为不同的节点设置不同的css样式
            if (d.type === 0) {
              return 'nodeOrange'
            } else if (d.type === 1) {
              return 'nodeBlue'
            } else if (d.type === 2) {
              return 'nodeRed'
            }
          })
          .attr('id', (d, i) => {
//            为每个节点设置不同的id
            return 'node' + i
          })
          .on('touchmove', (d, i) => {
//            设置鼠标监听时间，当移动端手指移动时,设置关系文本透明度
            edgesText.style('fill-opacity', function (edge) {
              if (edge.source === d || edge.target === d) {
                return 1.0
              } else {
                return 0
              }
            })
            /**
             * 改本svg的层级，这个主要是因为在svg中z-index是无效的，svg根据绘制的先后顺序，后绘制的排在最上面，就像贴纸，
             * 后贴的会盖住前面贴的。所以我们希望在被选中时，能够把节点和节点对应的文字提到最上一层，我们就可以通过d3来选择到点击的对象，然后通过raise方法来提到最上一层
             * 下同
             */
            d3.select('#node' + i).raise()
            d3.select('#nodetext' + i).raise()
          })
          .on('touchend', (d, i) => {
//            手指移开后，所有关系文本设置透明度为1
            edgesText.style('fill-opacity', function (edge) {
              return 1.0
            })
          })
          .on('mousedown', (d, i) => {
            edgesText.style('fill-opacity', function (edge) {
              if (edge.source === d || edge.target === d) {
                return 1.0
              } else {
                return 0
              }
            })
            d3.select('#node' + i).raise()
            d3.select('#nodetext' + i).raise()
          })
          .on('mouseout', (d, i) => {
            console.log('tttt')
            edgesText.attr('fill-opacity', function (edge) {
              return 1
            })
          })
          .call(drag)//监听拖动事件
//
//
// 节点文字
        var nodeText = g.selectAll('.nodetext')
          .data(nodes)
          .enter()
          .append('text')
          .attr('text-anchor', 'middle')
          .attr('class', 'nodetext')
          .attr('id', (d, i) => {
            return 'nodetext' + i
          })
          .attr('x', function (d, i) {
            /**
             * 由于svg的text不能进行换行，所以下面文字使用了tspan进行换行操作
             */
              //正则表达式
            var reEn = /[a-zA-Z]+/g
            //如果全英文则不换行
            if (d.name.match(reEn)) {
              d3.select(this).append('tspan')
                .attr('class', 'nodetext')
                .attr('fill', '#ff7438')
                .text(function () { return d.name })
            } else if (d.name.length <= 4) {
              //文中小于4个字不换行
              d3.select(this).append('tspan')
                .attr('class', 'nodetext')
                .attr('fill', '#ff7438')
                .text(function () { return d.name })
            } else {
              if (d.name.length <= 8) {
                //中文小于八个字，则分段进行换行
                let top = d.name.substring(0, 4)
                let bot = d.name.substring(4, 8)
                //这里的this指代text dom，不懂的可以自行打印this查看
                d3.select(this).append('tspan')
                  .text(function () { return top })
                d3.select(this).append('tspan')
                  .attr('dy', '1.2em')//设置偏移
                  .text(function () { return bot })
              } else {
                //中文大于8个字，分段并用...代替后面的字符
                let top = d.name.substring(0, 4)
                let bot = d.name.substring(4, 7) + '...'
                d3.select(this).append('tspan')
                  .text(function () { return top })
                d3.select(this).append('tspan')
                  .attr('dy', '1.2em')
                  .text(function () { return bot })
              }
            }
          })
          .attr('cursor', 'default')//设置鼠标样式
          .on('touchmove', (d, i) => {
            edgesText.style('fill-opacity', function (edge) {
              if (edge.source === d || edge.target === d) {
                return 1.0
              } else {
                return 0
              }
            })
            //改本svg的层级
            d3.select('#node' + i).raise()
            d3.select('#nodetext' + i).raise()
          })
          .on('touchend', (d, i) => {
            edgesText.style('fill-opacity', function (edge) {
              return 1.0
            })
          })
          .on('mousedown', (d, i) => {
            edgesText.style('fill-opacity', function (edge) {
              if (edge.source === d || edge.target === d) {
                return 1.0
              } else {
                return 0
              }
            })
            d3.select('#node' + i).raise()
            d3.select('#nodetext' + i).raise()
          })
          .on('mouseout', (d, i) => {
            edgesText.style('fill-opacity', function (edge) {
              return 1.0
            })
          })
          .call(drag)
//        设置node和edge
        force.nodes(nodes)
          .force('link', d3.forceLink(links).distance(linkDistance).strength(0.1))
          .restart()
//        tick 表示当运动进行中每更新一帧时
        force.on('tick', function () {
//          //更新连接线的位置
          edgesLine.attr('d', function (d) {
            var path = 'M ' + d.source.x + ' ' + d.source.y + ' L ' + d.target.x + ' ' + d.target.y
            return path
          })

          //更新连接线上文字的位置
          edgesText.attr('x', function (d) {
            return (d.source.x + d.target.x) / 2
          })
          edgesText.attr('y', function (d) { return (d.source.y + d.target.y) / 2 })

          //更新结点图片和文字
          node.attr('cx', function (d) {
            return d.x
          })
          node.attr('cy', function (d) { return d.y })

          nodeText.attr('x', function (d) { return d.x })
          nodeText.attr('y', function (d) { return d.y })
          //动态更新sptan 的x的坐标
          nodeText.selectAll('tspan')
            .attr('x', function (d) {
              return d.x
            })
        })
      }
    },
    created () {
      this.$nextTick(() => {
        this.relation = JSON.parse('{"nodes":[{"name":"BetterVicky","type":0},{"name":"杭州市高新区（滨江）萧宏小额贷款有限公司","type":1},{"name":"浙江合德建设有限公司","type":1},{"name":"杭州萧山党山企业担保有限公司","type":1},{"name":"林爱萍","type":2},{"name":"申盛集团有限公司","type":2}],"links":[{"source":0,"target":1,"relation":"对外投资"},{"source":0,"target":2,"relation":"对外投资"},{"source":0,"target":3,"relation":"对外投资"},{"source":4,"target":0,"relation":"投资"},{"source":5,"target":0,"relation":"投资"}],"code":200,"message":"请求成功"}')
        console.log(this.relation)
        this.showd3()
      })
    }
  }
</script>

<style lang="stylus" rel="stylesheet/stylus">
  .container
    height 100%
    .exit
      position absolute
      top 20px
      left 20px
      width 60px
      height 25px
      background #0583f2
      border none
      border-radius 2px
      color #fff
      z-index 200
      &:hover
        background #1e82d9

  .labeltext
    font-size: 16px;
    font-family: SimSun;
    fill: #ff7438;

  .nodetext
    font-size: 12px;
    font-family: SimSun;
    fill: #fff;
    position relative

  .linetext
    font-size: 12px
    font-weight bold
    font-family: SimSun
    fill: #000 !important
    color #000
    fill-opacity: 1.0

  .svg
    position relative
    width 100%
    height 100%

  .edgepath
    pointer-events none
    stroke-width 0.5px

  .nodeOrange
    position relative
    fill #ff7438 !important

  .nodeRed
    position relative
    fill #ff4238 !important

  .nodeBlue
    position relative
    fill #029ed9 !important
</style>
