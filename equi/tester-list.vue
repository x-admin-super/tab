<template>
    <el-card shadow="never" class="aui-card--fill">
        <el-form :inline="true" class="demo-form-inline" ref="dataForm" :model="dataForm" @keyup.enter.native="getDataList(-1)">
            <el-form-item label="测试时间：" prop="rangeDate">
                <el-date-picker 
                style="width:280px" 
                v-model="rangeDate"
                value-format="yyyy-MM-dd"
                range-separator="至" 
                start-placeholder="开始日期" 
                end-placeholder="结束日期" 
                type="daterange">
                </el-date-picker>
            </el-form-item>
            <el-form-item >
                <el-button  class="btn_box" type="primary" @click="getDataToList">查询</el-button><el-button @click="resetForm('dataForm')">重置</el-button>
            </el-form-item>
        </el-form>
        <el-table 
        :data="dataList"
        border
        ref="singleTable"
        @selection-change="dataListSelectionChangeHandle"
        @sort-change="dataListSortChangeHandle"
        style="width: 100%;">
            <el-table-column prop="createDate" label="测试时间" header-align="center" align="center"></el-table-column>
            <el-table-column prop="eqCount" label="测试设备数量" header-align="center" align="center"></el-table-column>
            <el-table-column prop="normalCount" label="正常设备数量" header-align="center" align="center"></el-table-column>
            <el-table-column prop="abnormalCount" label="异常设备数量" header-align="center" align="center"></el-table-column>
            <el-table-column prop="status" label="操作" header-align="center" align="center" width="150">
                <template slot-scope="scope">
                    <el-button @click="dioInfoTog(scope.row)" type="text">详情</el-button>
                </template>
            </el-table-column>
        </el-table>
        <el-pagination
            :current-page="page"
            :page-sizes="[10, 20, 50, 100]"
            :page-size="limit"
            :total="total"
            layout="total, sizes, prev, pager, next, jumper"
            @size-change="pageSizeChangeHandle"
            @current-change="pageCurrentChangeHandle">
        </el-pagination>

        <!-- 详情 -->
        <el-dialog
        title="测试详情"
        :visible.sync="dioInfo"
        width="60%"
        :before-close="handleInfoClose">
        <h3>设备测试结果</h3>
        <el-row style="padding:10px 0;">
            <el-checkbox v-model="checked" @change="searDiaList">只显示异常设备</el-checkbox>
        </el-row>
        <el-table :data="tableData" tooltip-effect="dark" border style="width: 100%">
            <el-table-column prop="etProjectName" label="项目名称" header-align="center" align="center"></el-table-column>
            <el-table-column prop="cmName" label="小区名称" header-align="center" align="center"></el-table-column>
            <el-table-column prop="etBuildingNum" label="楼栋号" header-align="center" align="center"></el-table-column>
            <el-table-column prop="etUnit" label="单元号" header-align="center" align="center"></el-table-column>
            <el-table-column prop="etInstallLoc" label="安装位置" header-align="center" align="center"></el-table-column>
            <el-table-column prop="etEquipmentName" label="净化器名称" header-align="center" align="center"></el-table-column>
            <el-table-column prop="d" label="测试结果" header-align="center" align="center">
                <template slot-scope="scope">
                    <div v-if="scope.row.etIsRun == 1">正常</div>
                    <div v-if="scope.row.etIsRun == 0">不正常</div>
                </template>
            </el-table-column>
        </el-table>
        <el-pagination
            :current-page="diapage"
            :page-sizes="[10, 20, 50, 100]"
            :page-size="dialimit"
            :total="diatotal"
            layout="total, sizes, prev, pager, next, jumper"
            @size-change="diapageSizeChangeHandle"
            @current-change="diapageCurrentChangeHandle">
        </el-pagination>
        <div slot="footer" style="text-align:center">
            <el-button type="primary" @click="dioInfoClose">关闭</el-button>
        </div>
        </el-dialog>
    </el-card>
</template>
<style scoped>
    el-dialog__footer{
        text-align: center;
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
            getDataListURL: '/report/equipmenttestlog/page',	// 数据列表接口，API地址
            getDataListIsPage: true 	// 数据列表接口，是否需要分页？
        },
        dataLists:[
            {id:132},
            {id:132}
        ],
        tableData:null,
        checked:null,
        rangeDate:[],
        dataForm:{
          starDate:null,  
          endDate:null,
          // rangeDate:[]
        },
        diapage:1,
        dialimit:10,
        diatotal:null,
        dioInfo:false,
        etIsRun:null
        
      }
    },
    watch: {
      'rangeDate': {
        handler(val, oldVal) {
          if (val !== null) {
            this.dataForm.startDate = val[0]
            this.dataForm.endDate = val[1]
          } else {
            this.dataForm.startDate = null
            this.dataForm.endDate = null
          }
        },
        immediate: true,
        deep: true
      }
    },
    methods: {
        getDataToList(){
            // delete this.dataForm.rangeDate;
            this.getDataList();
        },
        searDiaList(val){
            this.checked = val;
            this.getDataListDia();
        },
        handleInfoClose(){
            this.dioInfo = false
        },
        getDataListDia(){ // 详情列表
            if(this.checked){this.etIsRun = 0} else {this.etIsRun = ''}
            this.$http.get('/equipmenttest/page',{params: {
                    page: this.diapage,
                    limit: this.dialimit,
                    logId: this.rowId.id,
                    etIsRun: this.etIsRun
                }
            }).then(({ data: res }) => {
                if(res.code == 0){
                    this.tableData = res.data.list;
                    this.diatotal = res.data.total;
                } 
            });
        },
        // 分页, 每页条数
        diapageSizeChangeHandle (val) {
            this.diapage = 1
            this.dialimit = val
            this.getDataListDia() // 弹框列表
        },
        diapageCurrentChangeHandle (val) {
            this.diapage = val
            this.getDataListDia() // 弹框列表
        },
        dioInfoClose(){
            this.dioInfo = false
        },
        dioInfoTog(row,blo){
            this.dioInfo = true
            this.rowId = row
            this.getDataListDia(row)
        },
        resetForm(formName) {
            this.rangeDate = [];
            this.$refs[formName].resetFields();
            this.getDataList(-1);
        }
    
    }
  }
</script>