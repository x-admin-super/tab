  <template>
  <div>
    <el-card style="margin-bottom:10px">
      <!-- 详情表头详细字段展示 -->
    <el-row style="padding:10px 0">
      <span style="font-size:16px;font-weight:bolder">设备信息</span>
    </el-row>
    <div class="tit-list">
      <el-row class="el-row" :gutter="20">
        <el-col :span="5">净化器名称：<span v-if="infoObj && infoObj.eiName">{{infoObj.eiName}}</span><span v-else>--</span></el-col>
        <el-col :span="5">净化器设备编号：<span v-if="infoObj && infoObj.eiNum">{{infoObj.eiNum}}</span><span v-else>--</span></el-col>
        <el-col :span="5">净化器处理风量：<span v-if="infoObj && infoObj.wind">{{infoObj.wind}}</span><span v-else>--</span></el-col>
        <el-col :span="5">净化器安装方式：<span v-if="infoObj && infoObj.inType == 1">集中式</span><span v-else-if="infoObj && infoObj.inType == 2">户式</span><span v-else>--</span></el-col>
        <el-col :span="4">项目名称：<span v-if="infoObj && infoObj.projectName">{{infoObj.projectName}}</span><span v-else>--</span></el-col>
      </el-row>
      <el-row :gutter="20">
        <el-col :span="5">小区名称：<span v-if="infoObj && infoObj.communityName">{{infoObj.communityName}}</span><span v-else>--</span></el-col>
        <el-col :span="5">楼栋号：<span v-if="infoObj && infoObj.eiBuildingNum">{{infoObj.eiBuildingNum}}</span><span v-else>--</span></el-col>
        <el-col :span="5">单元号：<span v-if="infoObj && infoObj.eiUnit">{{infoObj.eiUnit}}</span><span v-else>--</span></el-col>
        <el-col :span="5">安装位置：<span v-if="infoObj && infoObj.eiInstallLoc">{{infoObj.eiInstallLoc}}</span><span v-else>--</span></el-col>
        <el-col :span="4">处理户数：<span v-if="infoObj && infoObj.gatewayNum">{{infoObj.gatewayNum}}</span><span v-else>--</span></el-col>
      </el-row>
    </div>
    </el-card>
    <el-card>
    <!-- 历史数据 -->
    <el-row style="padding:10px 0;">
      <span style="font-size:16px;font-weight:bolder;color:#333">历史数据</span><span style="font-size:12px;margin-left:10px;color:#333;">(默认显示近3小时的数据)</span>
    </el-row>
    <div class="tit-list">
      <el-form :inline="true" class="demo-form-inline" ref="dataForm" :model="dataForm">
        <el-form-item prop="rangeDate">
          <el-date-picker
            v-model="rangeDate"
            type="daterange"
            value-format="yyyy-MM-dd"
            style="width:280px"
            :picker-options="pickerOptions"
            range-separator="至"
            start-placeholder="开始日期"
            end-placeholder="结束日期">
          </el-date-picker>
        </el-form-item>
        <el-form-item label="净化器处理效率:" prop="erProceRatio">
          <el-input style="width:100px" v-model="dataForm.erProceRatio" placeholder="最高值"></el-input>
        </el-form-item>
        <el-form-item label="油烟浓度:" prop="erPollCon">
          <el-input style="width:100px" v-model="dataForm.erPollCon"  placeholder="最低值"></el-input>
        </el-form-item>
        <el-form-item label="颗粒物:" prop="erPmCon">
          <el-input style="width:100px" v-model="dataForm.erPmCon"  placeholder="最低值"></el-input>
        </el-form-item>
        <el-form-item label="非甲烷总烃:" prop="erVocs">
          <el-input style="width:100px" v-model="dataForm.erVocs"  placeholder="最低值"></el-input>
        </el-form-item>
        <el-form-item>
          <el-button  class="btn_box" @click="getDataToList" type="primary">查询</el-button><el-button @click="resetForm('dataForm')">重置</el-button>
        </el-form-item>
      </el-form>
    </div>
    <div class="con-wap">
      <div class="con-lt">
        <el-table v-loading="loadingTab" 
        :data="tabData"
        border
        style="width: 100%;">
        <el-table-column prop="recordTime" label="时间" header-align="center"></el-table-column>
        <el-table-column prop="erProceRatio" label="净化器处理效率" header-align="center">
          <template slot-scope="scope">
            <div :class="[parseInt(scope.row.erProceRatio)  > parseInt(resData.list.baseProceRatio) ? 'font-red' :'']">{{scope.row.erProceRatio}}</div>
          </template>
        </el-table-column>
        <el-table-column prop="erPollCon" label="油烟浓度" header-align="center">
          <template slot-scope="scope">
            <div :class="[parseInt(scope.row.erPollCon)  > parseInt(resData.list.basePollCon) ? 'font-red' :'']">{{scope.row.erPollCon}}</div>
          </template>
        </el-table-column>
        <el-table-column prop="erPmCon" label="颗粒物" header-align="center">
          <template slot-scope="scope">
            <div :class="[parseInt(scope.row.erPmCon)  > parseInt(resData.list.basePmCon) ? 'font-red' :'']">{{scope.row.erPmCon}}</div>
          </template>
        </el-table-column>
        <el-table-column prop="erVocs" label="非甲烷总烃" header-align="center">
          <template slot-scope="scope">
            <div :class="[parseInt(scope.row.erVocs)  > parseInt(resData.list.baseVocs) ? 'font-red' :'']">{{scope.row.erVocs}}</div>
          </template>
        </el-table-column>
      </el-table>
      <el-pagination
            :current-page="page"
            :page-sizes="[100,150,200,250,300]"
            :page-size="100"
            :total="total"
            layout="total, sizes, prev, pager, next, jumper"
            @size-change="pageSizeChangeHandle"
            @current-change="pageCurrentChangeHandle">
        </el-pagination>
      </div>
      <div class="con-rt">
        <el-card v-loading="loadingLine" class="line-card" style="margin-bottom:10px;">
          <div id="linea" class="line-box"></div>
        </el-card>
        <el-card v-loading="loadingLine" class="line-card">
          <div id="lineb" class="line-box" ></div>
        </el-card>
      </div>
    </div>
  </el-card>
  </div>
</template>
<style scoped>
  .con-wap{
    overflow: hidden;
  }
  .font-red{
    color:#FA0404;
  }
  .con-wap::after{
    overflow: hidden;
    content:'';
    clear: both;
  }
  .con-lt{
    width:49%;
    float: left;
  }
  .con-rt{
    width:49%;
    float: right;
  }
  .line-box{
    height:300px;
  }
  .line-card{
    background:#F8F8F8;
  }
  .el-row{
    font-size:14px;
    margin-bottom: 10px;
  }
</style>

<script>
import mixinViewModule from '@/mixins/view-module'
  export default {
    mixins: [mixinViewModule],
    data() {
      return {
        mixinViewModuleOptions: {
            activatedIsNeed: true,	// 此页面是否在激活（进入）时，调用查询数据列表接口？
            getDataListURL: '/sys/equipmentinfo/record',	// 数据列表接口，API地址
            getDataListIsPage: true 	// 数据列表接口，是否需要分页？
        },
        pickerOptions: {
            disabledDate: this.disabledDate
        },
        loading: false,
        loadingLine:false,
        loadingTab:false,
        infoObj:null,
        lineInfoObj:null,
        diaData:null,
        tabData:null,
        rangeDate:[],
        dataForm:{
          erProceRatio:'',
          erPollCon:'',
          id:this.$route.query.riId,
          erPmCon:'',
          erVocs:'',
          startDate:'',
          endDate:'',
          // rangeDate:[]
        },
      }
    },
    watch: {
      'rangeDate': {
        handler(val, oldVal) {
          if (val) {
            this.dataForm.startDate = val[0]
            this.dataForm.endDate = val[1]
          } else {
            this.dataForm.startDate = null
            this.dataForm.endDate = null
          }
        },
        immediate: true,
        deep: true
      },
      lineInfoObj(){
        if (!this.lineInfoObj) {
          this.loadingLine = true;
        } else {
          this.loadingLine = false
        }
      },
      'resData.pageData.list':{
        handler(val, oldVal){
          this.tabData = this.resData.pageData.list;
          this.total = this.resData.pageData.total;
          if (!this.tabData) {
            this.loadingTab = true;
          } else {
            this.loadingTab = false
          }
        },
        deep:true
      }
    },
    created(){
      this.getInfo();
      this.getLineInfo();
      
    },
    methods:{
      getDataToList(){
          // delete this.dataForm.rangeDate;
          this.loadingTab = true; this.loadingLine = true;
          // this.lineInfoObj = null; this.dataList = null;
          this.getDataList();
          this.getLineInfo();
      },
      disabledDate(time){
          const curDate = (new Date()).getTime()
          const day = -30 * 24 * 3600 * 1000
          const dateRegion = curDate + day
          return time.getTime() < dateRegion || time.getTime() > new Date().getTime();
      },
      myEcharts(ya,xa){
        var myChart = this.$echarts.init(document.getElementById('linea'));
        var option = {
          title:{
            left: "center",
            subtext:'净化器处理效率（%）',
            subtextStyle: {
              color:'#333333',
              fontSize: 14
            }
          },  
          tooltip: {},
          legend: {
            show:false,
            data:['销量']
          },
          xAxis: {
            data: xa,
            nameTextStyle:{
              color:"green"
            },
            axisLine:{
              lineStyle:{
                color:'#333'
              }
            }
          },
          yAxis: {
            "axisLine":{       
              "show":false
            },
            "axisTick":{       
              "show":false
            },
            "splitLine": {     
              "show": true
            }
          },
          series: [{
            name: '净化器处理效率',
            type: 'line',
            data: ya
          }]
        };

        myChart.setOption(option);
      },
      myEchartst(ya,yb,yc,xa){
        var myChart = this.$echarts.init(document.getElementById('lineb'));
        var option = {
          title:{
            left: "center",
            subtext:'在线监测（mg/m³）',
            subtextStyle: {
              color:'#333333',
              fontSize: 14
            }
          },  
          tooltip: {},
          legend: {
            show:false,
            data:['销量']
          },
          xAxis: {
            data: xa,
            nameTextStyle:{
              color:"green"
            },
            axisLine:{
              lineStyle:{
                color:'#333'
              }
            }
          },
          yAxis: {
            "axisLine":{       
              "show":false
            },
            "axisTick":{       
              "show":false
            },
            "splitLine": {     
              "show": true
            }
          },
          series: [
            {
                name: '颗粒物浓度',
                type: 'line',
                stack: '总量',
                data: ya
            },
            {
                name: '油烟浓度',
                type: 'line',
                stack: '总量',
                data: yb
            },
            {
                name: '非甲烷总烃',
                type: 'line',
                stack: '总量',
                data: yc
            }
          ]
        };

        myChart.setOption(option);
      },
      resetForm(formName) {
        this.$refs[formName].resetFields();
        this.rangeDate = []; this.dataForm['startDate']='';this.dataForm['endDate'] = '';
        this.getDataList(-1);
        this.getLineInfo();
      },
      getInfo(){ // 设备信息
        let rowId = this.$route.query.riId;
        this.$http.get('/sys/equipmentinfo/'+ rowId).then(({ data: res }) => {
            if(res.code == 0 && res.data.length != 0){
                this.infoObj = res.data;
            } 
        });
      },
      getLineInfo(){ // 统计图
        var data = this.dataForm;
        // delete data.rangeDate
        this.$http.get('/sys/equipmentinfo/chart',{params:data}).then(({ data: res }) => {
            if(res.code == 0){
                this.lineInfoObj = res.data;

                var erPmCon = [],erPollCon = [],erProceRatio = [],erVocs = [],recordTime = [];
                for(let o in res.data){
                  erPmCon.push(res.data[o]['erPmCon']);erPollCon.push(res.data[o]['erPollCon']);erProceRatio.push(res.data[o]['erProceRatio']);erVocs.push(res.data[o]['erVocs']);recordTime.push(res.data[o]['recordTime']);
                }
                this.myEcharts(erProceRatio,recordTime); // 净化器处理效率（%）
                this.myEchartst(erPmCon,erPollCon,erVocs,recordTime); // 在线监测

            } 
        });
      }
    },
    mounted() {
      
    }
  }
</script>