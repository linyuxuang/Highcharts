# Highcharts
Highcharts图表滑动翻页加载


      

  <link rel="stylesheet" href="{{APP_CSS_PATH}}slider/style.css" />
  <link rel="stylesheet" href="{{APP_CSS_PATH}}shop/index.css" />
  <script src="{{APP_JS_PATH}}highchats/highcharts.js"></script>
  <script type="text/javascript" src="{{APP_JS_PATH}}common/common.js"></script>
  <script type="text/javascript" src="{{APP_JS_PATH}}slider/cookie.js"></script>


  <div class="charts_s"  id="container"></div>
  <div style="position: relative;display: none " id="scalepage_s" >
    <div style="position: absolute;top: -44px;">
      <span id="title0"></span>
      <div class="scale" id="bar0">
        <div id="thisWidth"></div>
        <span id="btn0"></span>
      </div>
    </div>
  </div>

  <script>
    $(function(){
      var charts;
      cpageIndex=1;
      cpageOffset=10;

      equipment_statistics_diagram(cpageIndex,cpageOffset,true);
    });
    function equipment_statistics_diagram(pageIndex,pageOffset,isdraw){
      var res = {arrName:[],arronlines:[],arrofflines:[]};
      $.ajax({
        url: "{{APP_SERVER}}Devcpe/groupcount",
        type: "POST",
        dataType: "JSON",
        data: {
          "buss_id":"{buss_id}",
          "groupid":0,
          "pageIndex":pageIndex,
          "pageOffset":pageOffset
        },
        success: function (r) {
          if(r.code == 0){
            for(var i=0;i< Math.max(pageOffset,r.data.list.length);i++){
              if(r.data.list[i]){
                res.arrName.push(r.data.list[i].name_d);
                res.arronlines.push(parseInt(r.data.list[i].onlines));
                res.arrofflines.push(parseInt(r.data.list[i].offlines))
              }
            }
            if(isdraw)
              drawflag(res);
            else
              setflag(res);
            if($("#scalepage_s") && $("#scalepage_s").css("display")=='none')
              scalepage(r.data.total,cpageOffset,1089);
          }else{
            if(r.code == 405500){
              window.location.href = "{{APP_SERVER}}Home/shop"
            }else{
              show_message("error",r.msg,1000);
            }
          }

        }
      });
    }
    function setflag(data){
      charts.xAxis[0].setCategories(data.arrName,false);
      charts.series[0].setData(data.arronlines,false);
      charts.series[1].setData(data.arrofflines,false);
      charts.redraw();
    }

    function drawflag(data){
      charts = Highcharts.chart('container',{
        chart: {
          type: 'column',
          animation:{
            duration:1000
          }
        },
        xAxis: {
          categories: data.arrName
        },
        yAxis: {
          min: 0,
          title:{
            text:''
          }
        },
        legend: {
          align: 'right',
          verticalAlign: 'top',
          y:-15
        },
        tooltip: {
          headerFormat:'<span style="">{point.key}</span><br/><span style="color: #508DFE;">设备总数: </span><span>{point.total}</span><br/>',
          pointFormat: '<span style="color:{series.color}">{series.name}</span>: <b>{point.y}</b> ({point.percentage:.0f}%)<br/>',
          shared: true
        },

        plotOptions: {
          column: {
            stacking: 'normal',
            dataLabels: {
              enabled: false
            },
            pointWidth:$('#container').width() /20,
          },
          series: {
            animation: {
              duration: 2000,
              easing: 'easeOutBounce'
            }
          }
        },
        title: {
          text: '设备分布统计图',
          x: -460 ,
          y:10,
          fontSize: 5
        },
        credits:{
          enabled: true,
          position: {
            align: 'right',
            x: -10,
            y: -10
          },
          text: ""
        },
        series: [
          {
            name: '在线设备',
            data: data.arronlines,
            color:"#14C84B"
          }, {
            name: '离线设备',
            data:data.arrofflines,
            color:"#FE5858"
          }]
      });
    }


    function scalepage(total,pageOffset,lengths) {
      $("#scalepage_s").css("display","block");
      var p = 1;//页码
      var n = Math.ceil(total/pageOffset);//总页数
      var ssstep = lengths/n;//滑块长度
      $(".scale").css("width",lengths);
      $(".scale>span").css("width",ssstep);
      scale = function (btn, bar, title) {
        this.btn = document.getElementById(btn);
        this.bar = document.getElementById(bar);
        this.title = document.getElementById(title);
        this.step = this.bar.getElementsByTagName("DIV")[0];
        this.init();
      };
      scale.prototype = {
        init: function () {
          var f = this;
          var g = document;
          var b = window;
          var m = Math;
          this.title.innerHTML = total + '/1';
          //onmousedown 当在段落上按下鼠标按钮时执行
          f.btn.onmousedown = function (e) {
            var x = (e || b.event).clientX;
            var l = this.offsetLeft;
            var max = f.bar.offsetWidth - this.offsetWidth;
            //onmousemove 属性在鼠标指针移动到元素上时触发。
            g.onmousemove = function (e) {
              var thisX = (e || b.event).clientX;
              var to = m.min(max, m.max(-2, l + (thisX - x)));
              p = m.round(to / ssstep);
              f.btn.style.left = to + 'px';
              f.ondrag(p, to);
              b.getSelection ? b.getSelection().removeAllRanges() : g.selection.empty();
            };
            //onmouseup在用户鼠标按键被松开时执行
            g.onmouseup = function(){
              this.onmousemove=null;
              var left = p * ssstep;
              //alert(left);
              $("#btn0").animate({left:left});
              //alert(left)
              cpageIndex = p + 1;
              equipment_statistics_diagram(cpageIndex,cpageOffset,false);
            };

          };
        },
        ondrag: function (pos, x) {
          // alert("ooo")
          this.step.style.width = Math.max(0, x) + 'px';
          this.title.innerHTML = total + '/' + (pos+1) / 1 + '';
        }
      };
      new scale('btn0', 'bar0', 'title0');
    }
  </script>
