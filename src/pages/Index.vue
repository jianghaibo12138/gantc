<template>
  <q-page class="flex flex-center">
    <canvas ref="main-canvas" id="canvas" :width="width" :height="height" class="canvas"></canvas>
  </q-page>
</template>

<script>
export default {
  name: 'PageIndex',
  data () {
    return {
      canvas: null,
      context: null,
      xOffset: 50,
      yOffset: 50,
      width: 1200,
      height: 800,
      xStep: 10,
      xStepTimes: 0,
      yStep: 50,
      axisWidth: 5,
      unitCodes: ['拖拉机', '挖掘机', '推土机', '三蹦子'],
      planDetails: [{
        startTime: new Date('2020-01-01 00:00:00').getTime() / 1000,
        endTime: new Date('2020-02-01 00:00:00').getTime() / 1000,
        unitCode: '拖拉机'
      }, {
        startTime: new Date('2020-02-01 00:01:00').getTime() / 1000,
        endTime: new Date('2020-03-01 00:00:00').getTime() / 1000,
        unitCode: '挖掘机'
      }, {
        startTime: new Date('2020-04-01 00:01:00').getTime() / 1000,
        endTime: new Date('2020-05-01 00:00:00').getTime() / 1000,
        unitCode: '挖掘机'
      }, {
        startTime: new Date('2020-06-01 00:01:00').getTime() / 1000,
        endTime: new Date('2020-07-01 00:00:00').getTime() / 1000,
        unitCode: '挖掘机'
      }, {
        startTime: new Date('2020-03-01 00:01:00').getTime() / 1000,
        endTime: new Date('2020-04-01 00:00:00').getTime() / 1000,
        unitCode: '推土机'
      }, {
        startTime: new Date('2020-04-01 00:01:00').getTime() / 1000,
        endTime: new Date('2020-05-01 00:00:00').getTime() / 1000,
        unitCode: '三蹦子'
      }],
      minMaxTime: {
        min: new Date('2020-01-01 00:00:00').getTime() / 1000,
        max: new Date('2020-12-30 23:59:59').getTime() / 1000
      },
      yAxisInfo: {},
      timePerStep: 0,
      pointTopBottomOffset: 20,
      parentOffset: {}
    }
  },
  beforeCreate () {
    console.log(document.body.clientWidth, document.body.clientHeight)
    this.width = document.body.clientWidth * 0.8
    this.height = document.body.clientHeight * 0.8
  },
  created () {
  },
  async mounted () {
    // await this.planGenerate()
    await this.paramsInit()
    await this.canvasInit()
    await this.axisInit()
    await this.planInit()
    this.eventListener()
    console.log(this.timePerStep, this.parentOffset)
  },
  methods: {
    async paramsInit () {
      this.yStep = this.height / this.unitCodes.length
      this.parentOffset = this.offset(this.$refs['main-canvas'])
    },
    async canvasInit () {
      this.canvas = this.$refs['main-canvas']
      if (!this.canvas.getContext) {
        // drawing code here
      }
      this.ctx = this.canvas.getContext('2d')
    },
    async axisInit () {
      this.ctx.font = '15px serif'
      this.ctx.fillText('O(0, 0)', 0, this.yOffset * 0.8)

      // xAxis
      this.ctx.beginPath()
      this.ctx.moveTo(this.xOffset, this.yOffset)
      this.ctx.lineTo(this.xOffset, this.height)
      this.ctx.closePath()
      this.ctx.stroke()
      // yAxis
      this.ctx.beginPath()
      this.ctx.moveTo(this.xOffset, this.yOffset)
      this.ctx.lineTo(this.width, this.yOffset)
      this.ctx.closePath()
      this.ctx.stroke()

      // x arrow
      this.ctx.beginPath()
      this.ctx.moveTo(this.width, this.yOffset)
      this.ctx.lineTo(this.width - 20, this.xOffset * 0.8)
      this.ctx.closePath()
      this.ctx.stroke()

      // y arrow
      this.ctx.beginPath()
      this.ctx.moveTo(this.xOffset, this.height)
      this.ctx.lineTo(this.yOffset * 0.8, this.height - 20)
      this.ctx.closePath()
      this.ctx.stroke()

      // x step
      let x = this.xOffset, count = 0
      while (x <= (this.width - this.xOffset)) {
        let xPoint = 0
        if (count === 0) {
          xPoint = x
        } else {
          xPoint = x + this.xStep
          x += this.xStep
        }
        this.ctx.beginPath()
        this.ctx.moveTo(xPoint, this.yOffset)
        if (count % 5 === 0) {
          this.ctx.lineTo(xPoint, this.yOffset + this.axisWidth * 2)
          if (count !== 0) {
            this.ctx.fillText(count.toString(), xPoint - 5, this.yOffset * 0.8)
          }
        } else {
          this.ctx.lineTo(xPoint, this.yOffset + this.axisWidth)
        }
        this.ctx.closePath()
        this.ctx.stroke()
        count += 1
      }
      this.xStepTimes = count
      // console.log(count, 'count', (this.minMaxTime.max - this.minMaxTime.min))
      this.timePerStep = (this.minMaxTime.max - this.minMaxTime.min) / count

      // y step
      let y = this.yOffset, index = 0
      while (y < (this.height - this.yOffset)) {
        this.ctx.fillText(this.unitCodes[index], 0, y + this.yStep / 2)
        this.yAxisInfo[this.unitCodes[index]] = {
          x: 0,
          y: y + this.yStep / 2
        }
        index += 1
        this.ctx.beginPath()
        this.ctx.setLineDash([8, 8])
        this.ctx.moveTo(this.xOffset, y + this.yStep)
        this.ctx.lineTo(this.yOffset + this.width, y + this.yStep)
        // this.ctx.closePath()
        this.ctx.stroke()
        this.ctx.setLineDash([])
        y += this.yStep
      }
      console.log(this.yAxisInfo, this.timePerStep)
    },
    async planInit () {
      this.planDetails.forEach(plan => {
        const startTime = plan.startTime - this.minMaxTime.min,
          endTime = plan.endTime - this.minMaxTime.min
        const startPointX = (startTime / this.timePerStep * this.xStep) + this.xOffset,
          endPointX = (endTime / this.timePerStep * this.xStep) + this.xOffset
        const pointY = this.yAxisInfo[plan.unitCode].y - this.yStep / 2
        // console.log(startPointX, pointY, endPointX - startPointX)
        this.ctx.fillStyle = '#409EFF'
        this.ctx.strokeStyle = 'red'
        this.ctx.fillRect(startPointX, pointY + this.pointTopBottomOffset, endPointX - startPointX, this.yStep - this.pointTopBottomOffset * 2)
        this.ctx.strokeRect(startPointX, pointY + this.pointTopBottomOffset, endPointX - startPointX, this.yStep - this.pointTopBottomOffset * 2)
        this.ctx.fill()
        this.yAxisInfo[plan.unitCode].startPointX = startPointX
        this.yAxisInfo[plan.unitCode].startPointY = pointY + this.pointTopBottomOffset
        this.yAxisInfo[plan.unitCode].height = this.yStep - this.pointTopBottomOffset * 2
        this.yAxisInfo[plan.unitCode].width = (endPointX - startPointX)
      })
    },
    eventListener () {
      this.canvas.onmousemove = (e) => {
        const realX = e.clientX - this.parentOffset.left - this.xOffset
        const realY = e.clientY - this.parentOffset.top - this.yOffset
        const result = this.inBox(realX * 7.5, realY * 7.5)
        console.log(realX, realY, result)
      }
    },
    offset (curEle) {
      let totalLeft = null, totalTop = null, par = curEle.offsetParent
      // 首先把自己本身的进行累加
      totalLeft += curEle.offsetLeft
      totalTop += curEle.offsetTop

      // 只要没有找到body，我们就把父级参照物的边框和偏移量累加
      while (par) {
        if (navigator.userAgent.indexOf('MSIE 8.0') === -1) {
          // 不是标准的ie8浏览器，才进行边框累加
          // 累加父级参照物边框
          totalLeft += par.clientLeft
          totalTop += par.clientTop
        }
        // 累加父级参照物本身的偏移
        totalLeft += par.offsetLeft
        totalTop += par.offsetTop
        par = par.offsetParent
      }
      return { left: totalLeft, top: totalTop }
    },
    inBox (realX, realY) {
      // todo 像素换算, 判断鼠标在矩形中
      // console.log(this.yAxisInfo)
      for (const key in this.yAxisInfo) {
        const p = this.yAxisInfo[key]
        console.log(p, realX, realY)
        const ltx = p.startPointX
        const lty = p.startPointY

        const rtx = ltx + p.width
        // const rbx = rtx
        // const lbx = ltx
        const lby = lty + p.height

        if (realX >= ltx && realX <= rtx && realY <= lty && realY <= lby) {
          return p
        }
      }
      return null
    }
  }
}
</script>
