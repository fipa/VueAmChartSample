<template>
  <div id="app">

    <div class="container vh-100" id="app">
      <nav data-th-replace="layout :: nav"></nav>

      <div id="chartdiv"></div>
    </div>
    
  </div>
</template>

<script>

import * as am5 from "@amcharts/amcharts5";
import * as am5xy from "@amcharts/amcharts5/xy";
import am5themes_Animated from "@amcharts/amcharts5/themes/Animated";

export default{
  data(){
    return {
      employees: 
      [
        {
          "employeeName": "John",
          "tasks": [
            {
              "projectNumber": "1010",
              "startTime": "2022-11-18T09:15:57.267369129",
              "endTime": "2022-11-18T09:18:47.267369129",
              "employeeName": "John"
            },
            {
              "projectNumber": "1010",
              "startTime": "2022-11-18T10:25:57.267369129",
              "endTime": "2022-11-18T13:18:47.267369129",
              "employeeName": "John"
            }
          ]
        },
        {
          "employeeName": "Jane",
          "tasks": [
            {
              "projectNumber": "1011",
              "startTime": "2022-11-18T09:25:57.267369129",
              "endTime": "2022-11-18T10:18:47.267369129",
              "employeeName": "Jane"
            },
            {
              "projectNumber": "1011",
              "startTime": "2022-11-18T10:35:57.267369129",
              "endTime": "2022-11-18T11:18:47.267369129",
              "employeeName": "Jane"
            }
          ]
        },
        {
          "employeeName": "Elmo",
          "tasks": [
            {
              "projectNumber": "1012",
              "startTime": "2022-11-18T09:45:57.267369129",
              "endTime": "2022-11-18T11:18:47.267369129",
              "employeeName": "Elmo"
            },
            {
              "projectNumber": "1012",
              "startTime": "2022-11-18T11:15:57.267369129",
              "endTime": "2022-11-18T11:58:47.267369129",
              "employeeName": "Elmo"
            }
          ]
        },

      ]      
      ,
      projects: 
        {
          "1010": {
            "projectNumber": "1010",
            "projectName": "PROJECT-1",
          },
          "1011": {
            "projectNumber": "1011",
            "projectName": "PROJECT-3",
          },
          "1012": {
            "projectNumber": "1012",
            "projectName": "PROJECT-2",

          }
        }      
      ,
      am5Root: null
    }
    
  },
  mounted: function() {

      var self=this;
      var employees = self.employees;
      var projects = self.projects;
      am5.ready(function() {
        self.am5Root = am5.Root.new("chartdiv", {
          tooltipContainerBounds: {
            top: 200,
            right: 100,
            bottom: 200,
            left: 100
          }
        });
        
        var root = self.am5Root;
        root.setThemes([
          am5themes_Animated.new(root)
        ]);
        
        var chart = root.container.children.push(am5xy.XYChart.new(root, {
          panX: true,
          panY: false,
          wheelX: "panX",
          wheelY: "zoomX",
          layout: root.verticalLayout
        }));
        
        var yAxis = chart.yAxes.push(
          am5xy.CategoryAxis.new(root, {
            categoryField: "category",
            renderer: am5xy.AxisRendererY.new(root, {
              minGridDistance : 10
            }),
          })
        );

        let yRenderer = yAxis.get("renderer");
        yRenderer.labels.template.setAll({
          fontSize: "0.7em"
        });        

        var xAxisData = [];
        var yAxisData = [];

        for (var employee of employees) {

          for (var task of employee.tasks) {
            if (yAxisData.findIndex(x => x.category === employee.employeeName) == -1) {
              yAxisData.push( { category: employee.employeeName });
            }

            xAxisData.push({
              category: employee.employeeName,
              start: new Date(task.startTime).getTime(),
              end: new Date(task.endTime).getTime(),
              columnSettings: {
                stroke: am5.color('#000000'),
                strokeOpacity: 0.75
              },
              task: projects[task.projectNumber].projectName,
            });
            
          }
        }
        yAxis.data.setAll(yAxisData);

        var xAxis = chart.xAxes.push(
          am5xy.DateAxis.new(root, {
            baseInterval: { timeUnit: "minute", count: 1 },
            renderer: am5xy.AxisRendererX.new(root, {})
          })
        );
        let xRenderer = xAxis.get("renderer");
        xRenderer.labels.template.setAll({
          fontSize: "0.7em"
        });
        xRenderer.ticks.template.setAll({
          location: 0,
          multiLocation: 0,
          visible: true
        });

        var tooltip = am5.Tooltip.new(root, {
          keepTargetHover: true
        });
        tooltip.get("background").setAll({
          fill: am5.color('#f4f4f4'),
          fillOpacity: 1,
          stroke: am5.color('#000000'),
          strokeOpacity: 1
        });
        tooltip.label.setAll({
          fill: am5.color('#222222')
        });

        var series = chart.series.push(am5xy.ColumnSeries.new(root, {
          xAxis: xAxis,
          yAxis: yAxis,
          openValueXField: "start",
          valueXField: "end",
          categoryYField: "category",
          tooltip: tooltip
        }));

        series.columns.template.set("tooltipHTML",
                        "<span style='font-size:0.8em'>" +
                        "<a href='http://www.google.com' target='_blank'>Link 1</a> | <a href='http://www.google.com' target='_blank'>Link 2</a>" +
                        "</span>"
        );

        series.columns.template.setAll({
          templateField: "columnSettings"
        });

        series.bullets.push(function () {
          return am5.Bullet.new(root, {
            locationX: 0.7,
            sprite: am5.Label.new(root, {
              text: "{task}",
              fontSize: "0.7em",
              centerY: am5.p50,
              centerX: am5.p100,
              populateText: true
            })
          });
        });


        var cellSize = 25;
        series.events.on("datavalidated", function(ev) {

          var series = ev.target;
          var chart = series.chart;
          var xAxis = chart.xAxes.getIndex(0);

          var chartHeight = (1 + yAxisData.length) * cellSize + xAxis.height() + chart.get("paddingTop", 0) + chart.get("paddingBottom", 0);
          chart.root.dom.style.height = chartHeight + "px";
        });

        series.data.setAll(xAxisData);

        chart.set("scrollbarX", am5.Scrollbar.new(root, { orientation: "horizontal" }));
        series.appear();
        chart.appear(1000, 100);
        
        });

    
  }
};

</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

#chartdiv {
    width: 100%;
    height: 500px;
  }
</style>
