<style>
* {
  margin: 0;
  padding: 0;
  overflow: unset;
}

html,
body,
.el-container,
.el-aside {
  height: 100%;
}

html,
body {
  background-color: #f8f8f8;
}

.el-container {
  padding: 10px;
}

.el-aside,
.el-main,
.el-footer {
  padding: 5px;
}

.el-main,
.el-footer {
  height: auto;
}

.el-aside {
  margin-right: 5px;
  overflow: unset;
}

/* .el-card__body{
  padding: 0
} */

.el-textarea {
  border-bottom: 1px solid #ebeef5;
}

textarea {
  border: none;
}

.el-textarea__inner {
  border: none;
}

.buttonGroup {
  padding-top: 10px;
  display: flex;
  justify-content: flex-end;
}
.el-dialog--center .el-dialog__body {
  text-align: center;
}

/* button {
  width: 100px;
  height: 30px;
  margin-left: 10px;
} */

.el-switch {
  color: lightgrey;
  margin-bottom: 10px;
}

.el-switch__label {
  font-size: 15px;
}
.el-card__body {
  padding: 10px;
}
.el-form-item__label,
.el-tabs__item {
  font-weight: 600;
}
.el-form {
  margin-bottom: 10px;
}
.el-form-item {
  margin-bottom: 0;
}
#shockCharts,
#slopeCharts,
#tempCharts,
#pressureCharts {
  padding-bottom: 30px;
  box-sizing: border-box;
}
/* .el-carousel__item h3 {
  color: #475669;
  font-size: 14px;
  opacity: 0.75;
  line-height: 150px;
  margin: 0;
} */

/* .el-carousel__item:nth-child(2n) {
  background-color: #99a9bf;
}

.el-carousel__item:nth-child(2n + 1) {
  background-color: #d3dce6;
} */
.el-carousel__item {
  /* background-color: #f7f7f7; */
}
.el-carousel__indicators .el-carousel__button {
  background-color: #ccc;
}
/* .el-carousel__indicators.is-active .el-carousel__button{
  background-color: #000;
} */
</style>

<template>

<el-container>
    <el-aside style="" width="240px">
        <el-card style="height:100%">
            <el-form :label-position="labelPosition" label-width="80px" :model="formLabelAlign">
                <el-form-item label="选择接口">
                    <el-select v-model="selectedValue" placeholder="请选择" @change="handleClick">
                        <el-option v-for="(item,index) in ports" :key="index" :label="item.comName" :value="index">
                        </el-option>
                    </el-select>
                </el-form-item>
                <!-- <el-form-item label="波特率">
                    <el-input v-model="baudRate"></el-input>
                </el-form-item> -->
                <el-button type="success" style="width:100%;margin:10px 0" round @click='sendorder'>查询</el-button>
                <el-button type="success" style="width:48%;margin: 0" round @click='saveDatas'>保存</el-button>
                <el-button type="success" style="width:48%;margin: 0" round @click='openFile'>打开</el-button>
                <el-form-item label="开始时间">
                   <el-date-picker
                    style="width:100%"
                    v-model="startTime"
                    type="datetime"
                    placeholder="选择日期时间">
                  </el-date-picker>
                </el-form-item>
                <el-form-item label="结束时间">
                   <el-date-picker
                      style="width:100%"
                      v-model="endTime"
                      type="datetime"
                      placeholder="选择日期时间">
                    </el-date-picker>
                </el-form-item>
            </el-form>
            <el-button type="success" style="width:100%;margin: 0;margin-top: 10px;" round @click='exportExcel'>导出数据</el-button>
        </el-card>
    </el-aside>
    <el-container style="padding:0">
        <el-main>
            <el-card style="height:100%;box-sizing: border-box;">
                <el-tabs v-model="activeName"  stretch>
                    <!-- <el-tab-pane label="数据" name="second">
                      
                    </el-tab-pane> -->
                    <el-tab-pane label="图表" name="first">
                       <el-carousel :autoplay="false" trigger="click" :height="chartheight + 'px'">
                        <el-carousel-item :key="1">
                          <div id="shockCharts" :style="{'height':chartheight+'px'}"></div>
                        </el-carousel-item>
                         <el-carousel-item :key="2">
                          <div id="slopeCharts" :style="{'height':chartheight+'px'}"></div>
                        </el-carousel-item>
                        <el-carousel-item :key="3">
                          <div id="tempCharts" :style="{'height':chartheight+'px'}"></div>
                        </el-carousel-item>
                        <el-carousel-item :key="5">
                          <div id="pressureCharts" :style="{'height':chartheight+'px'}"></div>
                        </el-carousel-item>
                      </el-carousel>
                    </el-tab-pane>
                </el-tabs>
            </el-card>
        </el-main>
       
    </el-container>
    <el-dialog title="提示" :visible.sync="centerDialogVisible" width="30%" center :show-close="false" :close-on-click-modal="false" :close-on-press-escape="false">
      <span style="text-align:center">
        数据采集中
      </span>
    </el-dialog>
    <div style="display:none">
      <div id="shockCharts1" :style="{'height':chartheight+'px','width':chartheight+'px'}"></div>
      <div id="slopeCharts1" :style="{'height':chartheight+'px','width':chartheight+'px'}"></div>
      <div id="tempCharts1" :style="{'height':chartheight+'px','width':chartheight+'px'}"></div>
      <div id="pressureCharts1" :style="{'height':chartheight+'px','width':chartheight+'px'}"></div>         
    </div>
</el-container>


</template>

<script>
import Serialport from "serialport";
import echarts from "echarts";
import { setInterval, setTimeout, clearTimeout, clearInterval } from "timers";
import XLSX from "xlsx";
import fs from "fs";
import xl from "excel4node";
console.log(fs);
// import {ByteBuffer}  from './bytebuffer.js';
// debugger;
var ByteBuffer = require("bytebuffer");
// import covButton from './button'
export default {
  name: "test",
  data() {
    return {
      text: "",
      labelPosition: "top",
      formLabelAlign: {
        name: "",
        region: "",
        type: ""
      },
      selectedValue: 0,
      ports: [],
      baudRate: "9600",
      activeName: "first",
      textarea: "",
      userMySQL: false,
      source: [],
      maxLengthNum: 100,
      chartheight: document.documentElement.clientHeight - 150,
      animation: true,
      int: null,
      startTime: "",
      endTime: "",
      shock_quantity_x: [],
      shock_quantity_x1: [],
      shock_quantity_xobj: {},
      shock_quantity_xdate: [],
      shock_quantity_y: [],
      shock_quantity_y1: [],
      shock_quantity_yobj: {},
      shock_quantity_ydate: [],
      shock_quantity_z: [],
      shock_quantity_z1: [],
      shock_quantity_zobj: {},
      shock_quantity_zdate: [],
      slope_angle_x: [],
      slope_angle_x1: [],
      slope_angle_xobj: {},
      slope_angle_xdate: [],
      slope_angle_y: [],
      slope_angle_y1: [],
      slope_angle_yobj: {},
      slope_angle_ydate: [],
      temp: [],
      temp1: [],
      tempobj: {},
      tempdate: [],
      pressure1: [],
      pressure11: [],
      pressure1obj: {},
      pressure1date: [],
      pressure2: [],
      pressure2obj: {},
      pressure2date: [],
      chartSDate: [],
      chartSDate1: [],
      allDate: [],
      _shockCharts: null,
      _slopeCharts: null,
      _tempCharts: null,
      _pressureCharts: null,
      _shockCharts1: null,
      _slopeCharts1: null,
      _tempCharts1: null,
      _pressureCharts1: null,
      zIArr: [],
      hexCharCodeStrs: "",
      givestring: "",
      setId: "",
      setIn: "",
      centerDialogVisible: false
    };
  },
  computed: {
    //计算属性混入Vue实例
    // 加速度
    shockCharts() {
      return {
        tooltip: {
          trigger: "axis",
          triggerOn: "click",
          position: function(a, b, c, d) {
            // chartsClick(b);
          }
        },
        legend: {
          data: ["冲击加速度x(g)", "冲击加速度y(g)", "冲击加速度z(g)"]
        },
        grid: {
          left: "12%"
        },
        toolbox: {
          show: true,
          feature: {
            dataView: { show: true, readOnly: false },
            magicType: { show: true, type: ["stack", "tiled", "line", "bar"] },
            saveAsImage: { show: true }
          },
          top: 20
        },
        xAxis: {
          type: "category",
          // type:'time',
          boundaryGap: false,
          axisPointer: {
            value: this.chartSDate[0],
            snap: true,
            lineStyle: {
              color: "#004E52",
              opacity: 0.5,
              width: 2
            },
            label: {
              show: true,
              formatter: function(params) {
                return echarts.format.formatTime(
                  "yyyy-MM-dd hh:mm:ss",
                  params.value
                );
              },
              backgroundColor: "#004E52"
            },
            handle: {
              show: true,
              color: "#004E52"
            }
          },
          data: this.chartSDate
        },
        yAxis: {
          type: "value",
          min: function(value) {
            return value.min - 0.01;
          },
          max: function(value) {
            return value.max + 0.01;
          }
        },
        dataZoom: [
          {
            type: "slider",
            show: true,
            start: 0,
            end: 100,
            handleSize: 8
          },
          {
            type: "inside",
            start: 90,
            end: 100
          }
        ],
        series: [
          {
            name: "冲击加速度x(g)",
            type: "line",
            smooth: true,
            data: this.shock_quantity_x
          },
          {
            name: "冲击加速度y(g)",
            type: "line",
            smooth: true,
            data: this.shock_quantity_y
          },
          {
            name: "冲击加速度z(g)",
            type: "line",
            smooth: true,
            data: this.shock_quantity_z
          }
        ]
      };
      // statChart.setOption(option);
    },

    // 倾斜度
    slopeCharts() {
      return {
        tooltip: {
          trigger: "axis",
          triggerOn: "click",
          position: function(a, b, c, d) {
            // chartsClick(b);
          }
        },
        legend: {
          data: ["倾斜度x(°)", "倾斜度y(°)"]
        },
        grid: {
          left: "12%"
        },
        toolbox: {
          show: true,
          feature: {
            dataView: { show: true, readOnly: false },
            magicType: { show: true, type: ["stack", "tiled", "line", "bar"] },
            saveAsImage: { show: true }
          },
          top: 20
        },
        xAxis: {
          type: "category",
          // type:'time',
          boundaryGap: false,
          axisPointer: {
            value: this.chartSDate[0],
            snap: true,
            lineStyle: {
              color: "#004E52",
              opacity: 0.5,
              width: 2
            },
            label: {
              show: true,
              formatter: function(params) {
                return echarts.format.formatTime(
                  "yyyy-MM-dd hh:mm:ss",
                  params.value
                );
              },
              backgroundColor: "#004E52"
            },
            handle: {
              show: true,
              color: "#004E52"
            }
          },
          data: this.chartSDate
        },
        yAxis: {
          type: "value",
          min: function(value) {
            return value.min - 1;
          },
          max: function(value) {
            return value.max + 1;
          }
        },
        dataZoom: [
          {
            type: "slider",
            show: true,
            start: 0,
            end: 100,
            handleSize: 8
          },
          {
            type: "inside",
            start: 90,
            end: 100
          }
        ],
        series: [
          {
            name: "倾斜度x(°)",
            type: "line",
            smooth: true,
            data: this.slope_angle_x
          },
          {
            name: "倾斜度y(°)",
            type: "line",
            smooth: true,
            data: this.slope_angle_y
          }
        ]
      };
      // slope_charts.setOption(option);
    },

    //温度
    tempCharts() {
      return {
        tooltip: {
          trigger: "axis",
          triggerOn: "click",
          position: function(a, b, c, d) {
            // chartsClick(b);
          }
        },
        legend: {
          data: ["温度(℃)"]
        },
        grid: {
          left: "12%"
        },
        toolbox: {
          show: true,
          feature: {
            dataView: { show: true, readOnly: false },
            magicType: { show: true, type: ["stack", "tiled", "line", "bar"] },
            saveAsImage: { show: true }
          },
          top: 20
        },
        xAxis: {
          type: "category",
          // type:'time',
          boundaryGap: false,
          axisPointer: {
            value: this.chartSDate[0],
            snap: true,
            lineStyle: {
              color: "#004E52",
              opacity: 0.5,
              width: 2
            },
            label: {
              show: true,
              formatter: function(params) {
                return echarts.format.formatTime(
                  "yyyy-MM-dd hh:mm:ss",
                  params.value
                );
              },
              backgroundColor: "#004E52"
            },
            handle: {
              show: true,
              color: "#004E52"
            }
          },
          data: this.chartSDate
        },
        yAxis: {
          type: "value",
          min: function(value) {
            return value.min - 1;
          },
          max: function(value) {
            return value.max + 1;
          }
        },
        dataZoom: [
          {
            type: "slider",
            show: true,
            start: 0,
            end: 100,
            handleSize: 8
          },
          {
            type: "inside",
            start: 90,
            end: 100
          }
        ],
        series: [
          {
            name: "温度(℃)",
            type: "line",
            smooth: true,
            data: this.temp
          }
        ]
      };
      // temp_charts.setOption(option);
    },
    //气压
    pressureCharts() {
      return {
        tooltip: {
          trigger: "axis",
          triggerOn: "click",
          position: function(a, b, c, d) {
            // chartsClick(b);
          }
        },
        legend: {
          data: ["气压(KPa)"]
        },
        grid: {
          left: "12%"
        },
        toolbox: {
          show: true,
          feature: {
            dataView: { show: true, readOnly: false },
            magicType: { show: true, type: ["stack", "tiled", "line", "bar"] },
            saveAsImage: { show: true }
          },
          top: 20
        },
        xAxis: {
          type: "category",
          // type:'time',
          boundaryGap: false,
          axisPointer: {
            value: this.chartSDate[0],
            snap: true,
            lineStyle: {
              color: "#004E52",
              opacity: 0.5,
              width: 2
            },
            label: {
              show: true,
              formatter: function(params) {
                return echarts.format.formatTime(
                  "yyyy-MM-dd hh:mm:ss",
                  params.value
                );
              },
              backgroundColor: "#004E52"
            },
            handle: {
              show: true,
              color: "#004E52"
            }
          },
          data: this.chartSDate
        },
        yAxis: {
          type: "value",
          min: function(value) {
            return value.min - 1;
          },
          max: function(value) {
            return value.max + 1;
          }
        },
        dataZoom: [
          {
            type: "slider",
            show: true,
            start: 0,
            end: 100,
            handleSize: 8
          },
          {
            type: "inside",
            start: 90,
            end: 100
          }
        ],
        series: [
          {
            name: "气压(KPa)",
            type: "line",
            smooth: true,
            data: this.pressure1
          }
        ]
      };
      // pressure1_charts.setOption(option);
    },

    shockCharts1() {
      return {
        tooltip: {
          trigger: "axis",
          triggerOn: "click"
        },
        legend: {
          data: ["冲击加速度x(g)", "冲击加速度y(g)", "冲击加速度z(g)"]
        },
        grid: {
          left: "12%"
        },
        toolbox: {
          show: true,
          feature: {
            dataView: { show: true, readOnly: false },
            magicType: { show: true, type: ["stack", "tiled", "line", "bar"] },
            saveAsImage: { show: true }
          },
          top: 20
        },
        xAxis: {
          type: "category",
          // type:'time',
          boundaryGap: false,
          axisPointer: {
            value: this.chartSDate[0],
            snap: true,
            lineStyle: {
              color: "#004E52",
              opacity: 0.5,
              width: 2
            },
            label: {
              show: true,
              formatter: function(params) {
                return echarts.format.formatTime(
                  "yyyy-MM-dd hh:mm:ss",
                  params.value
                );
              },
              backgroundColor: "#004E52"
            }
          },
          data: this.chartSDate1
        },
        yAxis: {
          type: "value",
          min: function(value) {
            return value.min - 0.01;
          },
          max: function(value) {
            return value.max + 0.01;
          }
        },
        dataZoom: [
          {
            type: "slider",
            show: true,
            start: 0,
            end: 100,
            handleSize: 8
          },
          {
            type: "inside",
            start: 90,
            end: 100
          }
        ],
        series: [
          {
            name: "冲击加速度x(g)",
            type: "line",
            smooth: true,
            data: this.shock_quantity_x1
          },
          {
            name: "冲击加速度y(g)",
            type: "line",
            smooth: true,
            data: this.shock_quantity_y1
          },
          {
            name: "冲击加速度z(g)",
            type: "line",
            smooth: true,
            data: this.shock_quantity_z1
          }
        ]
      };
    },

    // 倾斜度
    slopeCharts1() {
      return {
        tooltip: {
          trigger: "axis",
          triggerOn: "click"
        },
        legend: {
          data: ["倾斜度x(°)", "倾斜度y(°)"]
        },
        grid: {
          left: "12%"
        },
        toolbox: {
          show: true,
          feature: {
            dataView: { show: true, readOnly: false },
            magicType: { show: true, type: ["stack", "tiled", "line", "bar"] },
            saveAsImage: { show: true }
          },
          top: 20
        },
        xAxis: {
          type: "category",
          // type:'time',
          boundaryGap: false,
          axisPointer: {
            value: this.chartSDate[0],
            snap: true,
            lineStyle: {
              color: "#004E52",
              opacity: 0.5,
              width: 2
            },
            label: {
              show: true,
              formatter: function(params) {
                return echarts.format.formatTime(
                  "yyyy-MM-dd hh:mm:ss",
                  params.value
                );
              },
              backgroundColor: "#004E52"
            }
          },
          data: this.chartSDate1
        },
        yAxis: {
          type: "value",
          min: function(value) {
            return value.min - 1;
          },
          max: function(value) {
            return value.max + 1;
          }
        },
        dataZoom: [
          {
            type: "slider",
            show: true,
            start: 0,
            end: 100,
            handleSize: 8
          },
          {
            type: "inside",
            start: 90,
            end: 100
          }
        ],
        series: [
          {
            name: "倾斜度x(°)",
            type: "line",
            smooth: true,
            data: this.slope_angle_x1
          },
          {
            name: "倾斜度y(°)",
            type: "line",
            smooth: true,
            data: this.slope_angle_y1
          }
        ]
      };
    },

    //温度
    tempCharts1() {
      return {
        tooltip: {
          trigger: "axis",
          triggerOn: "click"
        },
        legend: {
          data: ["温度(℃)"]
        },
        grid: {
          left: "12%"
        },
        toolbox: {
          show: true,
          feature: {
            dataView: { show: true, readOnly: false },
            magicType: { show: true, type: ["stack", "tiled", "line", "bar"] },
            saveAsImage: { show: true }
          },
          top: 20
        },
        xAxis: {
          type: "category",
          boundaryGap: false,
          axisPointer: {
            value: this.chartSDate[0],
            snap: true,
            lineStyle: {
              color: "#004E52",
              opacity: 0.5,
              width: 2
            },
            label: {
              show: true,
              formatter: function(params) {
                return echarts.format.formatTime(
                  "yyyy-MM-dd hh:mm:ss",
                  params.value
                );
              },
              backgroundColor: "#004E52"
            }
          },
          data: this.chartSDate1
        },
        yAxis: {
          type: "value",
          min: function(value) {
            return value.min - 1;
          },
          max: function(value) {
            return value.max + 1;
          }
        },
        dataZoom: [
          {
            type: "slider",
            show: true,
            start: 0,
            end: 100,
            handleSize: 8
          },
          {
            type: "inside",
            start: 90,
            end: 100
          }
        ],
        series: [
          {
            name: "温度(℃)",
            type: "line",
            smooth: true,
            data: this.temp1
          }
        ]
      };
    },
    //气压
    pressureCharts1() {
      return {
        tooltip: {
          trigger: "axis",
          triggerOn: "click"
        },
        legend: {
          data: ["气压(KPa)"]
        },
        grid: {
          left: "12%"
        },
        toolbox: {
          show: true,
          feature: {
            dataView: { show: true, readOnly: false },
            magicType: { show: true, type: ["stack", "tiled", "line", "bar"] },
            saveAsImage: { show: true }
          },
          top: 20
        },
        xAxis: {
          type: "category",
          // type:'time',
          boundaryGap: false,
          axisPointer: {
            value: this.chartSDate[0],
            snap: true,
            lineStyle: {
              color: "#004E52",
              opacity: 0.5,
              width: 2
            },
            label: {
              show: true,
              formatter: function(params) {
                return echarts.format.formatTime(
                  "yyyy-MM-dd hh:mm:ss",
                  params.value
                );
              },
              backgroundColor: "#004E52"
            }
          },
          data: this.chartSDate1
        },
        yAxis: {
          type: "value",
          min: function(value) {
            return value.min - 1;
          },
          max: function(value) {
            return value.max + 1;
          }
        },
        dataZoom: [
          {
            type: "slider",
            show: true,
            start: 0,
            end: 100,
            handleSize: 8
          },
          {
            type: "inside",
            start: 90,
            end: 100
          }
        ],
        series: [
          {
            name: "气压(KPa)",
            type: "line",
            smooth: true,
            data: this.pressure11
          }
        ]
      };
    },

    maxLength: {
      get: function() {
        return this.maxLengthNum; //获取的时候直接获取值
      },
      set: function(value) {
        if (Object.is(value * 1, NaN)) {
          this.$notify({
            title: "警告",
            message: "请输入数字",
            type: "warning",
            offset: 10
          });
        } else {
          this.maxLengthNum = value;
        }
      }
    }
  },
  mounted() {
    //生命周期
    var _this = this;
    Serialport.list((err, ports) => {
      this.ports = ports;
      _this.handleClick();
      console.log(err, ports, "ports");
      ports.forEach(ele => {
        console.log(ele, "port");
      });
    });

    this.$nextTick(function() {
      _this.init();

      // _this.drawLine() ;
    });
  },
  methods: {
    init() {
      const self = this;

      setTimeout(() => {
        window.onresize = function() {
          // self._shockCharts.resize
          self.chartheight = document.documentElement.clientHeight - 150;
          // self._shockCharts.resize()
          self._shockCharts.resize();
          self._slopeCharts.resize();
          self._tempCharts.resize();
          self._pressureCharts.resize();
        };
      }, 20);
      self._shockCharts = echarts.init(document.getElementById("shockCharts"));
      self._slopeCharts = echarts.init(document.getElementById("slopeCharts"));
      self._tempCharts = echarts.init(document.getElementById("tempCharts"));
      self._pressureCharts = echarts.init(
        document.getElementById("pressureCharts")
      );
      self._shockCharts1 = echarts.init(
        document.getElementById("shockCharts1")
      );
      self._slopeCharts1 = echarts.init(
        document.getElementById("slopeCharts1")
      );
      self._tempCharts1 = echarts.init(document.getElementById("tempCharts1"));
      self._pressureCharts1 = echarts.init(
        document.getElementById("pressureCharts1")
      );

      self._shockCharts.setOption(this.shockCharts);
      self._slopeCharts.setOption(this.slopeCharts);
      self._tempCharts.setOption(this.tempCharts);
      self._pressureCharts.setOption(this.pressureCharts);

      self._shockCharts1.setOption(this.shockCharts1);
      self._slopeCharts1.setOption(this.slopeCharts1);
      self._tempCharts1.setOption(this.tempCharts1);
      self._pressureCharts1.setOption(this.pressureCharts1);
    },

    initchartsDate() {
      var _this = this;
      _this.shock_quantity_x = [];
      _this.shock_quantity_x1 = [];
      _this.shock_quantity_xobj = {};
      _this.shock_quantity_xdate = [];
      _this.shock_quantity_y = [];
      _this.shock_quantity_y1 = [];
      _this.shock_quantity_yobj = {};
      _this.shock_quantity_ydate = [];
      _this.shock_quantity_z = [];
      _this.shock_quantity_z1 = [];
      _this.shock_quantity_zobj = {};
      _this.shock_quantity_zdate = [];
      _this.slope_angle_x = [];
      _this.slope_angle_x1 = [];
      _this.slope_angle_xobj = {};
      _this.slope_angle_xdate = [];
      _this.slope_angle_y = [];
      _this.slope_angle_y1 = [];
      _this.slope_angle_yobj = {};
      _this.slope_angle_ydate = [];
      _this.temp = [];
      _this.temp1 = [];
      _this.tempobj = {};
      _this.tempdate = [];
      _this.pressure1 = [];
      _this.pressure11 = [];
      _this.pressure1obj = {};
      _this.pressure1date = [];
      _this.pressure2 = [];
      _this.pressure2obj = {};
      _this.pressure2date = [];
      _this.chartSDate = [];
      _this.chartSDate1 = [];
      _this.allDate = [];
    },
    exportExcel() {
      var self = this;
      var startTime = new Date(self.startTime).format("yyyy-MM-dd hh:mm:ss");
      var endTime = new Date(self.endTime).format("yyyy-MM-dd hh:mm:ss");

      console.log(startTime, endTime);

      if (
        startTime == "NaN-aN-aN aN:aN:aN" ||
        endTime == "NaN-aN-aN aN:aN:aN"
      ) {
        self.$message({ message: "请选择开始时间和结束时间", type: "warning" });
        return;
      }
      var datas = [];
      self.chartSDate1 = [];
      self.shock_quantity_x1 = [];
      self.shock_quantity_y1 = [];
      self.shock_quantity_z1 = [];
      self.slope_angle_x1 = [];
      self.slope_angle_y1 = [];
      self.temp1 = [];
      self.pressure11 = [];
      self.allDate.forEach(ele => {
        if (ele["时间"] >= startTime && ele["时间"] <= endTime) {
          datas.push(ele);
          for (var k in ele) {
            switch (k) {
              case "时间":
                self.chartSDate1.push(ele[k]);
                break;
              case "冲击加速度x":
                self.shock_quantity_x1.push(ele[k]);
                break;
              case "冲击加速度y":
                self.shock_quantity_y1.push(ele[k]);
                break;
              case "冲击加速度z":
                self.shock_quantity_z1.push(ele[k]);
                break;
              case "倾斜度x":
                self.slope_angle_x1.push(ele[k]);
                break;
              case "倾斜度y":
                self.slope_angle_y1.push(ele[k]);
                break;
              case "温度":
                self.temp1.push(ele[k]);
                break;
              case "气压1":
                self.pressure11.push(ele[k]);
                break;
            }
          }
        }
      });

      self._shockCharts1.setOption(this.shockCharts1);
      self._slopeCharts1.setOption(this.slopeCharts1);
      self._tempCharts1.setOption(this.tempCharts1);
      self._pressureCharts1.setOption(this.pressureCharts1);

      var data = {
        时间: "时间",
        冲击加速度x: "冲击加速度x(g)",
        冲击加速度y: "冲击加速度y(g)",
        冲击加速度z: "冲击加速度z(g)",
        倾斜度x: "倾斜度x(°)",
        倾斜度y: "倾斜度y(°)",
        温度: "温度(℃)",
        气压1: "气压(KPa)",
        气压2: "气压2(KPa)"
      };

      datas.unshift(data);
      console.log(datas);
      // return
      var shockURL = self._shockCharts1.getDataURL({
        pixelRation: 1,
        backgroundColor: "#fff",
        excludeComponents: ["toolbox", "dataZoom"]
      });
      var slopeURL = self._slopeCharts1.getDataURL({
        pixelRation: 1,
        backgroundColor: "#fff",
        excludeComponents: ["toolbox", "dataZoom"]
      });
      var tempURL = self._tempCharts1.getDataURL({
        pixelRation: 1,
        backgroundColor: "#fff",
        excludeComponents: ["toolbox", "dataZoom"]
      });
      var pressureURL = self._pressureCharts1.getDataURL({
        pixelRation: 1,
        backgroundColor: "#fff",
        excludeComponents: ["toolbox", "dataZoom"]
      });

      var shockbase64 = shockURL.replace(/^data:image\/\w+;base64,/, ""); //去掉图片base64码前面部分data:image/png;base64
      var slopebase64 = slopeURL.replace(/^data:image\/\w+;base64,/, ""); //去掉图片base64码前面部分data:image/png;base64
      var tempbase64 = tempURL.replace(/^data:image\/\w+;base64,/, ""); //去掉图片base64码前面部分data:image/png;base64
      var pressurebase64 = pressureURL.replace(/^data:image\/\w+;base64,/, ""); //去掉图片base64码前面部分data:image/png;base64

      var shockBuffer = new Buffer(shockbase64, "base64"); //把base64码转成buffer对象，
      var slopeBuffer = new Buffer(slopebase64, "base64"); //把base64码转成buffer对象，
      var tempBuffer = new Buffer(tempbase64, "base64"); //把base64码转成buffer对象，
      var pressureBuffer = new Buffer(pressurebase64, "base64"); //把base64码转成buffer对象，
      self.$message({ message: __dirname + " " + __filename });
      fs.writeFile(__dirname + "/shockURL.png", shockBuffer, function(err) {
        if (!err) {
          fs.writeFile(__dirname + "/slopeURL.png", slopeBuffer, function(err) {
            if (!err) {
              fs.writeFile(__dirname + "/tempURL.png", tempBuffer, function(
                err
              ) {
                if (!err) {
                  fs.writeFile(
                    __dirname + "/pressureURL.png",
                    pressureBuffer,
                    function(err) {
                      if (!err) {
                        console.log("写入成功！");
                        createExcel(datas);
                      }
                    }
                  );
                }
              });
            }
          });

          function createExcel(datas) {
            var wb = new xl.Workbook();
            var ws = wb.addWorksheet("数据");
            var ws1 = wb.addWorksheet("shock");
            var ws2 = wb.addWorksheet("slope");
            var ws3 = wb.addWorksheet("temp");
            var ws4 = wb.addWorksheet("pressure");

            var style = wb.createStyle({
              font: {
                color: "#000000",
                size: 12
              }
              // numberFormat: "$#,##0.00; ($#,##0.00); -"
            });

            datas.forEach((ele, index) => {
              for (var k in ele) {
                switch (k) {
                  case "时间":
                    setCell(ws, index + 1, 1, ele[k]);
                    break;
                  case "冲击加速度x":
                    setCell(ws, index + 1, 2, ele[k]);
                    break;
                  case "冲击加速度y":
                    setCell(ws, index + 1, 3, ele[k]);
                    break;
                  case "冲击加速度z":
                    setCell(ws, index + 1, 4, ele[k]);
                    break;
                  case "倾斜度x":
                    setCell(ws, index + 1, 5, ele[k]);
                    break;
                  case "倾斜度y":
                    setCell(ws, index + 1, 6, ele[k]);
                    break;
                  case "温度":
                    setCell(ws, index + 1, 7, ele[k]);
                    break;
                  case "气压1":
                    setCell(ws, index + 1, 8, ele[k]);
                    break;
                }
              }
            });
            function setCell(ws, row, col, val) {
              // console.log(val);
              var value;
              if (col == 1 || row == 1) {
                value = val.toString();
                ws
                  .cell(row, col)
                  .string(value)
                  .style(style);
              } else {
                value = parseFloat(val);
                ws
                  .cell(row, col)
                  .number(value)
                  .style(style);
              }
            }

            WsaddImage(ws1, "shockURL.png");
            WsaddImage(ws2, "slopeURL.png");
            WsaddImage(ws3, "tempURL.png");
            WsaddImage(ws4, "pressureURL.png");

            console.log(wb);
            // wb.write("Excel.xlsx");
            wb.writeToBuffer().then(function(buffer) {
              // console.log(buffer);
              var tmpDown = new Blob([buffer], { type: "" }); //创建二进制对象写入转换好的字节流
              var a = document.createElement("a");
              var href = URL.createObjectURL(tmpDown); //创建对象超链接
              a.href = href; //绑定a标签
              a.download = "data.xlsx";
              document.body.appendChild(a);
              a.click(); //模拟点击实现下载
              setTimeout(function() {
                //延时释放
                URL.revokeObjectURL(tmpDown); //用URL.revokeObjectURL()来释放这个object URL
                document.body.removeChild(a);
              }, 100);
            });
          }

          function WsaddImage(ws, name) {
            self.$message({ message: __dirname + " " + __filename });
            ws.addImage({
              path: __dirname + "/" + name,
              type: "picture",
              position: {
                type: "oneCellAnchor",
                from: {
                  col: 1,
                  colOff: "0.5in",
                  row: 1,
                  rowOff: 0
                }
              }
            });
          }
        }
      });
    },
    // exportXlsx() {
    //   //导出数据
    //   var datas = this.allDate;
    //   var data = {
    //     时间: "时间",
    //     冲击加速度x: "冲击加速度x(g)",
    //     冲击加速度y: "冲击加速度y(g)",
    //     冲击加速度z: "冲击加速度z(g)",
    //     倾斜度x: "倾斜度x(°)",
    //     倾斜度y: "倾斜度y(°)",
    //     温度: "温度(℃)",
    //     气压1: "气压1(KPa)",
    //     气压2: "气压2(KPa)"
    //   };

    //   datas.unshift(data);

    //   function downloadExl(json, type) {
    //     var tmpDown; //导出的二进制对象
    //     var keyMap = []; //获取键
    //     for (var o in json[0]) {
    //       keyMap.push(o);
    //     }
    //     var tmpdata = []; //用来保存转换好的json
    //     json
    //       .map((v, i) =>
    //         keyMap.map((k, j) =>
    //           Object.assign(
    //             {},
    //             {
    //               //运用ES6内容
    //               v: v[k],
    //               position:
    //                 (j > 25 ? getCharCol(j) : String.fromCharCode(65 + j)) +
    //                 (i + 1)
    //             }
    //           )
    //         )
    //       )
    //       .reduce((prev, next) => prev.concat(next))
    //       .forEach(
    //         (v, i) =>
    //           (tmpdata[v.position] = {
    //             v: v.v
    //           })
    //       );

    //     var outputPos = Object.keys(tmpdata); //设置区域,比如表格从A1到D10
    //     var tmpWB = {
    //       SheetNames: ["mySheet"], //保存的表标题
    //       Sheets: {
    //         mySheet: Object.assign(
    //           {},
    //           tmpdata, //内容
    //           {
    //             "!ref": outputPos[0] + ":" + outputPos[outputPos.length - 1] //设置填充区域
    //           }
    //         )
    //       }
    //     };
    //     tmpDown = new Blob(
    //       [
    //         s2ab(
    //           XLSX.write(
    //             tmpWB,
    //             {
    //               bookType: type == undefined ? "xlsx" : type,
    //               bookSST: false,
    //               type: "binary"
    //             } //这里的数据是用来定义导出的格式类型
    //           )
    //         )
    //       ],
    //       {
    //         type: ""
    //       }
    //     ); //创建二进制对象写入转换好的字节流

    //     var a = document.createElement("a");
    //     var href = URL.createObjectURL(tmpDown); //创建对象超链接
    //     a.href = href; //绑定a标签
    //     a.download = "data.xlsx";
    //     document.body.appendChild(a);
    //     a.click(); //模拟点击实现下载
    //     setTimeout(function() {
    //       //延时释放
    //       URL.revokeObjectURL(tmpDown); //用URL.revokeObjectURL()来释放这个object URL
    //       document.body.removeChild(a);
    //     }, 100);
    //   }
    //   downloadExl(datas);
    //   this.$message("导出成功");

    //   // 将指定的自然数转换为26进制表示。映射关系：[0-25] -> [A-Z]。
    //   function getCharCol(n) {
    //     let temCol = "",
    //       s = "",
    //       m = 0;
    //     while (n > 0) {
    //       m = n % 26 + 1;
    //       s = String.fromCharCode(m + 64) + s;
    //       n = (n - m) / 26;
    //     }
    //     return s;
    //   }
    // },

    saveDatas() {
      var option = {
        date: this.chartSDate,
        shock_quantity_x: this.shock_quantity_x,
        shock_quantity_y: this.shock_quantity_y,
        shock_quantity_z: this.shock_quantity_z,
        slope_angle_x: this.slope_angle_x,
        slope_angle_y: this.slope_angle_y,
        temp: this.temp,
        pressure1: this.pressure1
      };
      var tmpDown = new Blob([s2ab(JSON.stringify(option))], { type: "" });
      var a = document.createElement("a");
      a.id = "hf";
      var href = URL.createObjectURL(tmpDown); //创建对象超链接
      a.href = href; //绑定a标签
      a.download = "data.txt";
      document.body.appendChild(a);
      a.click(); //模拟点击实现下载
      setTimeout(function() {
        //延时释放
        URL.revokeObjectURL(tmpDown); //用URL.revokeObjectURL()来释放这个object URL
        document.body.removeChild(a);
      }, 100);
    },
    openFile() {
      //上传文件打开视图
      var self = this;
      var input = document.createElement("input");
      input.type = "file";
      document.body.appendChild(input);
      input.click();
      input.addEventListener("change", function(e) {
        var files = e.target.files;
        var reader = new FileReader();
        reader.readAsText(files[0], "UTF-8");
        reader.onload = function(evt) {
          try {
            var option = JSON.parse(evt.target.result);
            (self.chartSDate = option.date),
              (self.shock_quantity_x = option.shock_quantity_x),
              (self.shock_quantity_y = option.shock_quantity_y),
              (self.shock_quantity_z = option.shock_quantity_z),
              (self.slope_angle_x = option.slope_angle_x),
              (self.slope_angle_y = option.slope_angle_y),
              (self.temp = option.temp),
              (self.pressure1 = option.pressure1);
            self._shockCharts.setOption(self.shockCharts);
            self._slopeCharts.setOption(self.slopeCharts);
            self._tempCharts.setOption(self.tempCharts);
            self._pressureCharts.setOption(self.pressureCharts);
          } catch (e) {
            console.log(e);
            self.$message({ message: "文本内容不对", type: "warning" });
          }
        };
        document.body.removeChild(input);
      });
    },
    drawLine(string) {
      //串口传输触发
      console.log("drawLine");
      console.log(string);
      var _this = this;
      // if (!_this.centerDialogVisible) {
      //   return;
      // }
      // var ssf =

      // console.log(ssf.toString("hex"))
      _this.setId++;
      chagnedialogText('数据采集中···  '+ _this.setId)
      // document.getElementsByClassName(
      //   "el-dialog__body"
      // )[0].children[0].innerHTML =
      //   "数据采集中···  " + _this.setId;

      console.log(_this.setId);

      // var dd = [];
      function getzarr(callback) {
        var startindex;
        var endindex;
        var strings = _this.givestring;
        startindex = strings.indexOf("7e");
        endindex = strings.indexOf("7e", startindex + 2) + 2;
        console.log(startindex, endindex, "dd");
        // console.log(startindex == )
        var dd = endindex - 4;
        console.log(startindex === dd ? true : false);
        var isj = startindex === dd ? true : false;
        if (isj) {
          startindex = startindex + 2;
          endindex = strings.indexOf("7e", startindex + 2) + 2;
        }
        console.log(startindex, endindex, "dd");
        if (startindex >= 0 && endindex >= 0) {
        } else {
          if (strings.indexOf("7d01") > -1) {
            startindex = strings.indexOf("7d01");
            endindex = strings.indexOf("7d02", startindex) + 4;
          }
        }

        if (startindex >= 0 && endindex >= 0 && endindex - startindex > 4) {
          console.log(startindex, endindex);
          var _thisz = strings.slice(startindex, endindex);
          _this.givestring = _this.givestring.slice(
            endindex,
            _this.givestring.length
          );
          console.log(_this.givestring.length);
          _this.zIArr.push(_thisz);
          getzarr(callback);
        } else {
          // if()
          callback();
        }
      }
      // _this.givestring += string.toString("hex");
      // _this.givestring = ssf.toString("hex")
      if (_this.setIn) {
        clearInterval(_this.setIn);
      }
      var i_i = 0;
      _this.setIn = setInterval(function() {
        i_i++;
        console.log(i_i, "i");
        if (i_i > 10) {
          if (_this.setIn) {
            clearInterval(_this.setIn);
          }
          if (_this.serialPort.isOpen) {
            _this.serialPort.close();
          }

          // fs.writeFileSync("aa.txt", _this.givestring);
          fs.writeFileSync("aa.txt", _this.givestring);
          chagnedialogText('数据正在解析中···')
          // debugger
          getzarr(function() {
            chagnedialogText('数据正在处理中···')
            console.log(_this.zIArr, "ziarr");
            _this.initchartsDate();
            // try {
            _this.zIArr.forEach(function(ele, i) {
              if (ele.indexOf("7d01") > -1 || ele.indexOf("7d02") > -1) {
                ele = ele.replace(/7d01/g, "7e");
                ele = ele.replace(/7d02/g, "7e");
              }
              getAllDate(ele, i);
            });
            for (var k in _this.shock_quantity_xobj) {
              _this.chartSDate.push(k);
              _this.shock_quantity_x.push(_this.shock_quantity_xobj[k]);
            }
            for (var k in _this.shock_quantity_xobj) {
              //以第一个时间为目标
              _this.shock_quantity_y.push(_this.shock_quantity_yobj[k]);
            }
            for (var k in _this.shock_quantity_xobj) {
              _this.shock_quantity_z.push(_this.shock_quantity_zobj[k]);
            }
            for (var k in _this.shock_quantity_xobj) {
              _this.slope_angle_x.push(_this.slope_angle_xobj[k]);
            }
            for (var k in _this.shock_quantity_xobj) {
              _this.slope_angle_y.push(_this.slope_angle_yobj[k]);
            }
            for (var k in _this.shock_quantity_xobj) {
              _this.temp.push(_this.tempobj[k]);
            }
            for (var k in _this.shock_quantity_xobj) {
              _this.pressure1.push(_this.pressure1obj[k]);
            }
            console.log(_this.pressure1obj,_this.pressure1)
            _this._shockCharts.setOption(_this.shockCharts);
            _this._slopeCharts.setOption(_this.slopeCharts);
            _this._tempCharts.setOption(_this.tempCharts);
            _this._pressureCharts.setOption(_this.pressureCharts);

            // _this._shockCharts1.setOption(_this.shockCharts1);
            // _this._slopeCharts1.setOption(_this.slopeCharts1);
            // _this._tempCharts1.setOption(_this.tempCharts1);
            // _this._pressureCharts1.setOption(_this.pressureCharts1);
            _this.centerDialogVisible = false;
            // } catch (e) {
            //   _this.$message({ message: e });
            // }
          });
        }
      }, 1000);

      function getAllDate(str, si) {
        var buffer = new ByteBuffer(0);
        buffer.prepend(new Buffer(str, "hex"), "hex");
        var header = buffer.readByte();
        var addr = buffer.readShort();
        buffer.readByte();
        var bufferlength = buffer.readShort();
        var size = buffer.readByte();
        var length = parseInt(bufferlength / size);
        console.log(si);
        // console.log(header, addr);
        // console.log(bufferlength);
        // console.log(length);
        // console.log(str)
        // console.log(buffer)
        // console.log(buffer.limit, "limit");
        for (var i = 0; i < length; i++) {
          try {
            var date = buffer.readBytes(6).toBuffer();
          } catch (e) {
            console.log(e, "date", i);
          }

          var dates = getDate(date);
          // console.log(date)
          // console.log(dates)
          // if(dates == '2018-08-31 16:16:27'){
          //   debugger;
          // }
          // console.log(dates);
          var fdates = new Date(dates).format("yyyy-MM-dd hh:00:00");
          // console.log(fdates,dates)
          if (fdates == "2018-08-29 16:52:18") {
            console.log(date, buffer, str);
            console.log(fdates, dates);
          }
          // if(fdates == '200-09-03 05:00:00'){
          //   console.log(date,buffer,str)
          //   console.log(fdates,dates)
          // }
          try {
            var _shock_quantity_x = buffer.readShort() / 100;
          } catch (e) {
            console.log(e, "ox", i, buffer.limit);
          }
          try {
            var _shock_quantity_y = buffer.readShort() / 100;
          } catch (e) {
            console.log(e, "oy", i, buffer.limit);
          }
          try {
            var _shock_quantity_z = buffer.readShort() / 100;
          } catch (e) {
            console.log(e, "oz", i, buffer.limit);
          }
          try {
            var _slope_angle_x = buffer.readByte();
          } catch (e) {
            console.log(e, "lx", i, buffer.limit);
          }
          try {
            var _slope_angle_y = buffer.readByte();
          } catch (e) {
            console.log(e, "ly", i);
          }
          try {
            var _temp = buffer.readByte();
          } catch (e) {
            console.log(e, "t", i);
          }
          try {
            var _pressure1 = buffer.readShort() / 10;
          } catch (e) {
            console.log(e, "p1", i);
          }
          try {
            var _pressure2 = buffer.readShort() / 10;
          } catch (e) {
            console.log(e, "p2", i);
          }
          // var _shock_quantity_x = buffer.readShort() / 100;
          // var _shock_quantity_y = buffer.readShort() / 100;
          // var _shock_quantity_z = buffer.readShort() / 100;
          // var _slope_angle_x = buffer.readByte();
          // var _slope_angle_y = buffer.readByte();
          // var _temp = buffer.readByte();
          // var _pressure1 = buffer.readShort() / 10;
          // var _pressure2 = buffer.readShort() / 10;
          if (dates.indexOf("NaN-aN-aN") == -1) {
            _this.shock_quantity_xobj = maxfun(
              _this.shock_quantity_xobj,
              fdates,
              _shock_quantity_x,
              8,
              -8
            );
            _this.shock_quantity_yobj = maxfun(
              _this.shock_quantity_yobj,
              fdates,
              _shock_quantity_y,
              8,
              -8
            );
            _this.shock_quantity_zobj = maxfun(
              _this.shock_quantity_zobj,
              fdates,
              _shock_quantity_z,
              8,
              -8
            );
            _this.slope_angle_xobj = maxfun(
              _this.slope_angle_xobj,
              fdates,
              _slope_angle_x,
              30,
              -30
            );
            _this.slope_angle_yobj = maxfun(
              _this.slope_angle_yobj,
              fdates,
              _slope_angle_y,
              30,
              -30
            );
            var temp1ave = avg(_this.temp1);
            var pressureave = avg(_this.pressure11);
            // console.log(pressureave-100,'111')
            _this.pressure1obj = maxfun(_this.pressure1obj, fdates, _pressure1,pressureave+200, temp1ave-200);
            _this.tempobj = maxfun(_this.tempobj, fdates, _temp,temp1ave+20,temp1ave-20);

            // _this.chartSDate1.push(dates)
            // _this.shock_quantity_x1.push(_shock_quantity_x);
            // _this.shock_quantity_y1.push(_shock_quantity_y);
            // _this.shock_quantity_z1.push(_shock_quantity_z);
            // _this.slope_angle_x1.push(_slope_angle_x);
            // _this.slope_angle_y1.push(_slope_angle_y);
            _this.temp1.push(_temp);
            _this.pressure11.push(_pressure1);

            var obj = {
              时间: dates,
              冲击加速度x: _shock_quantity_x,
              冲击加速度y: _shock_quantity_y,
              冲击加速度z: _shock_quantity_z,
              倾斜度x: _slope_angle_x,
              倾斜度y: _slope_angle_y,
              温度: _temp,
              气压1: _pressure1,
              气压2: _pressure2
            };
            _this.allDate.push(obj);
          }
        }
      }
      function getDate(datebuffer) {
        // debugger;
        if (datebuffer[0].toString().length == 1) {
          return "NaN-aN-aN aN:aN:aN";
        }
        var t =
          "20" +
          datebuffer[0] +
          "-" +
          datebuffer[1] +
          "-" +
          datebuffer[2] +
          " " +
          datebuffer[3] +
          ":" +
          datebuffer[4] +
          ":" +
          datebuffer[5];
        return new Date(t).format("yyyy-MM-dd hh:mm:ss");
      }

      function maxfun(obj, dates, val, maxval, minval) {
        // if(typeof man)
        var val = val == 0 ? val : parseFloat(val).toFixed(2)
        if (val > maxval || val < minval) {
          return obj;
        }
        if (obj[dates] != undefined && obj[dates] != "") {
          if (val < 0 && obj[dates] < 0) {
            if (Math.abs(val) > Math.abs(obj[dates])) {
              obj[dates] = val;
            }
          }
          if (val > 0 && obj[dates] < 0) {
            obj[dates] = val;
          }
          if (val > 0 && obj[dates] > 0) {
            if (val > obj[dates]) {
              obj[dates] = val;
            }
          }
        } else {
          obj[dates] = val;
        }
        return obj;
      }
    },
    handleClick() {
      //连接端口
      var _this = this;
      console.log(this.ports);
      console.log(this.ports[_this.selectedValue].comName);
      if (_this.serialPort) {
        if (_this.serialPort.isOpen) {
          _this.serialPort.close();
          _this.serialPort = "";
        }
      }

      _this.serialPort = new Serialport(
        this.ports[_this.selectedValue].comName,
        {
          baudRate: 115200,
          autoOpen: false
        }
      );
      //连接串口
      _this.serialPort.open(function(err) {
        console.log("IsOpen:", _this.serialPort.isOpen);
        if (!_this.serialPort.isOpen) {
          _this.centerDialogVisible = false;
          _this.$message({ message: "串口连接不上", type: "warning" });
        } else {
          _this.$message({ message: "串口已连接", type: "success" });
        }
      });
      //指令监听
      _this.serialPort.on("data", function(data) {
        // debugger;
        // console.log("data received: " + data);
        _this.drawLine(data);
      });
      //错误监听
      _this.serialPort.on("error", function(error) {
        console.log("error: " + error);
      });
    },

    sendorder() {
      //发送指令
      // console.log(this.startTime);
      // console.log(this.endTime);
      // if (!this.startTime && !this.endTime) {
      //   return;
      // }
      var _startTime = new Date('2018-06-09 09:09:09');
      var _endTime = new Date();

      var _startYear = parseInt(
        _startTime
          .getFullYear()
          .toString()
          .slice(2, 4)
      );
      var startYear = parseInt(
        _startTime
          .getFullYear()
          .toString()
          .slice(2, 4)
      ).toString(16);
      var _startmonth = parseInt(_startTime.getMonth() + 1);
      var startmonth = parseInt(_startTime.getMonth() + 1).toString(16);
      startmonth = startmonth.length == 1 ? "0" + startmonth : startmonth;
      var _startDay = _startTime.getDate();
      var startDay = _startTime.getDate().toString(16);
      startDay = startDay.length == 1 ? "0" + startDay : startDay;
      var _starthour = _startTime.getHours();
      var starthour = _startTime.getHours().toString(16);
      starthour = starthour.length == 1 ? "0" + starthour : starthour;
      var _startMinutes = _startTime.getMinutes();
      var startMinutes = _startTime.getMinutes().toString(16);
      startMinutes =
        startMinutes.length == 1 ? "0" + startMinutes : startMinutes;
      var _startSeconds = _startTime.getSeconds();
      var startSeconds = _startTime.getSeconds().toString(16);
      startSeconds =
        startSeconds.length == 1 ? "0" + startSeconds : startSeconds;

      var _endYear = parseInt(
        _endTime
          .getFullYear()
          .toString()
          .slice(2, 4)
      );
      var endYear = parseInt(
        _endTime
          .getFullYear()
          .toString()
          .slice(2, 4)
      ).toString(16);
      var _endmonth = parseInt(_endTime.getMonth() + 1);
      var endmonth = parseInt(_endTime.getMonth() + 1).toString(16);
      endmonth = endmonth.length == 1 ? "0" + endmonth : endmonth;
      var _endDay = _endTime.getDate();
      var endDay = _endTime.getDate().toString(16);
      endDay = endDay.length == 1 ? "0" + endDay : endDay;
      var _endhour = _endTime.getHours();
      var endhour = _endTime.getHours().toString(16);
      endhour = endhour.length == 1 ? "0" + endhour : endhour;
      var _endMinutes = _endTime.getMinutes();
      var endMinutes = _endTime.getMinutes().toString(16);
      endMinutes = endMinutes.length == 1 ? "0" + endMinutes : endMinutes;
      var _endSeconds = _endTime.getSeconds();
      var endSeconds = _endTime.getSeconds().toString(16);
      endSeconds = endSeconds.length == 1 ? "0" + endSeconds : endSeconds;

      console.log(
        _startYear,
        _startmonth,
        _startDay,
        _starthour,
        _startMinutes,
        _startSeconds,
        _endYear,
        _endmonth,
        _endDay,
        _endhour,
        _endMinutes,
        _endSeconds
      );

      var rbuf = new ByteBuffer(0);
      // 写入标识位
      rbuf.writeByte(0x7e);
      rbuf.writeShort(0x8900);
      rbuf.writeByte(0x30);
      rbuf.writeShort(0x000c);
      rbuf.writeByte(_startYear);
      rbuf.writeByte(_startmonth);
      rbuf.writeByte(_startDay);
      rbuf.writeByte(_starthour);
      rbuf.writeByte(_startMinutes);
      rbuf.writeByte(_startSeconds);
      rbuf.writeByte(_endYear);
      rbuf.writeByte(_endmonth);
      rbuf.writeByte(_endDay);
      rbuf.writeByte(_endhour);
      rbuf.writeByte(_endMinutes);
      rbuf.writeByte(_endSeconds);

      var getValidCode = function(buf) {
        var code = 0;
        for (var i = 1; i < buf.offset; i++) {
          code = code ^ buf.buffer[i];
        }
        return code;
      };
      var valid_code = getValidCode(rbuf).toString(16);
      console.log(valid_code, "dd");

      var _this = this;
      _this.centerDialogVisible = true;
      this.givestring = fs.readFileSync(
        "C:/Users/Administrator/Documents/Tencent Files/751476821/FileRecv/aa.txt",
        "utf-8"
      );
      _this.setId = 0;
      _this.zIArr = [];
      _this.initchartsDate();
      this.serialPort.isOpen ? null : _this.handleClick();

      var sendO =
        "7E890030000C" +
        startYear +
        startmonth +
        startDay +
        starthour +
        startMinutes +
        startSeconds +
        endYear +
        endmonth +
        endDay +
        endhour +
        endMinutes +
        endSeconds +
        valid_code +
        "7e";
      console.log(sendO, "33");
      this.serialPort.write(sendO, "hex", function(err) {
        if (err) {
          return console.log("Error on write: ", err.message);
        }
        console.log("message written");
      });
    }
  },

  components: {
    //自定义组件
    // covButton
  }
};
</script>
