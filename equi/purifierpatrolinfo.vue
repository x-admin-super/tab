<template>
<!-- 净化器巡查详情 -->
    <el-card shadow="never" class="aui-card--fill">
        <el-row style="padding:10px 0;">
            <el-checkbox v-model="checked" @change="equiChecked">只显示异常设备</el-checkbox>
        </el-row>
        <el-table 
        :data="dataList"
        border
        ref="singleTable"
        @selection-change="dataListSelectionChangeHandle"
        @sort-change="dataListSortChangeHandle"
        style="width: 100%;">
            <el-table-column prop="createDate" label="巡查时间" header-align="center" align="center"></el-table-column>
            <el-table-column prop="piCourtName" label="小区名称" header-align="center" align="center"></el-table-column>
            <el-table-column prop="piBuildingNum" label="楼栋号" header-align="center" align="center"></el-table-column>
            <el-table-column prop="piUnit" label="单元号" header-align="center" align="center"></el-table-column>
            <el-table-column prop="piInstallLoc" label="安装位置" header-align="center" align="center"></el-table-column>
            <el-table-column prop="piName" label="净化器名称" header-align="center" align="center"></el-table-column>
            <el-table-column prop="piStatus" label="巡查结果" header-align="center" align="center">
                <template slot-scope="scope">
                    <div v-if="scope.row.piStatus == 0">不正常</div>
                    <div v-if="scope.row.piStatus == 1">正常</div>
                </template>
            </el-table-column>
            <el-table-column prop="piPic" label="巡查照片" header-align="center" align="center">
                <template slot-scope="scope">
                    <img @click="handlePreviewSe(scope.row.piPic)" v-if="scope.row.piPic" :src="scope.row.piPic" alt="巡查照片">
                    <span v-else>--</span>
                </template>
            </el-table-column>
            <el-table-column prop="piConment" label="巡查说明" header-align="center" align="center"></el-table-column>
            <el-table-column prop="piIsCback" label="异常是否处理" header-align="center" align="center">
                <template slot-scope="scope">
                    <div v-if="scope.row.piIsCback == 0">是</div>
                    <div v-if="scope.row.piIsCback == 1">否</div>
                </template>
            </el-table-column>
            <el-table-column prop="status" label="处理结果" header-align="center" align="center" width="150">
                <template slot-scope="scope">
                    <el-button @click="dioInfoshow(scope.row)" type="text">处理结果</el-button>
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

        <!-- 处理结果 -->
        <el-dialog
        title="异常处理结果"
        :visible.sync="dioInfo"
        width="60%"
        :before-close="handleInfoClose">
        <el-form label-width="100px" class="demo-ruleForm">
            <el-form-item label="处理时间:" >
                <span v-if="dataList && dataList.piDoData">{{dataList.piDoData}}</span><span v-else>--</span>
            </el-form-item>
            <el-form-item label="处理照片:" >
                <el-row :gutter="10">
                    <el-col :span="12"><img @click="handlePreviewSe(dataList.piDoPic1)" v-if="dataList && dataList.piDoPic1" :src="dataList.piDoPic1" height="40" width="80"></el-col>
                    <el-col :span="12"><img @click="handlePreviewSe(dataList.piDoPic2)" v-if="dataList && dataList.piDoPic2" :src="dataList.piDoPic2" height="40" width="80"></el-col>
                </el-row>
                 <el-row :gutter="10">
                    <el-col :span="12"><img @click="handlePreviewSe(dataList.piDoPic3)" v-if="dataList && dataList.piDoPic3" :src="dataList.piDoPic3" height="40" width="80"></el-col>
                    <el-col :span="12"><img @click="handlePreviewSe(dataList.piDoPic4)" v-if="dataList && dataList.piDoPic4" :src="dataList.piDoPic4" height="40" width="80"></el-col>
                </el-row>
            </el-form-item>
            <el-form-item label="处理说明:" >
                <span v-if="dataList && dataList.piDoConment">{{dataList.piDoConment}}</span><span v-else>--</span>
            </el-form-item>
        </el-form>
        <div slot="footer" style="text-align:center">
            <el-button type="primary" @click="dioInfoClosed">关闭</el-button>
        </div>
        </el-dialog>

        <!-- 详情照片放大功能 -->
        <el-dialog :visible.sync="uploadImgVisible" width="50%">
            <img width="100%" :src="dialogImageUrl">
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
            getDataListURL: '/report/patrollog/page',	// 数据列表接口，API地址
            getDataListIsPage: true 	// 数据列表接口，是否需要分页？
        },
        dataForm:{
          eqType:this.$route.query.eqType,
          wheelId:this.$route.query.riId,
          eqExp:''
        },
        uploadImgVisible:false,
        dialogImageUrl:null,
        infoObj:null,
        checked:false,
        dioInfo:false
      }
    },
    methods: {
        // 照片放大功能
        handlePreviewSe(url){
            this.uploadImgVisible = true;
            this.dialogImageUrl = url;
        },
        dioInfoshow(row){
            this.dioInfo = true;
            // this.getresdata(row.id);
            
        },
        handleInfoClose(){
            this.dioInfo = false;
        },
        equiChecked(val){
            this.checked = val;
            if(val) {
                this.dataForm.eqExp = 0
            } else {
                this.dataForm.eqExp = ''
            }
            this.getDataList();
        },
        dioInfoClosed(){
            this.dioInfo = false;
        }
    }
  }
</script>