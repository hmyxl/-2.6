<template>
  <div class="MyTaskNew">
    <div class="contentTop" v-loading="loading3">
      <div class="paiTaskTitTab">
        <div class="paiTask pai active">新建任务</div>
      </div>
      <component v-bind:is="compArr.QuickCreateTaskComp" fileFormId="QuickCreateTaskComp"></component>
    </div>
    <div class="searchItem">
      <div>
        <el-select v-model="optionsValue" placeholder="请选择" @change="myTaskStyleChange($event)">
          <el-option
            v-for="item in optionsTask"
            :key="item.value"
            :label="item.label"
            :value="item.value">
          </el-option>
        </el-select>
      </div>
      <div>
        <el-select v-model="optionsValue2" placeholder="请选择" @change="taskTypeState($event)">
          <el-option
            v-for="item in optionsTask2"
            :key="item.value"
            :label="item.label"
            :value="item.value">
          </el-option>
        </el-select>
      </div>
      <div>
        <el-select v-model="optionsValue3" placeholder="请选择" @change="taskOfProject($event)">
          <el-option label="全部项目" value=""></el-option>
          <el-option
            v-for="item in projectList"
            :key="item.projectUID"
            :label="item.projectName"
            :value="item.projectName">
          </el-option>
        </el-select>
      </div>
      <div>
        <el-input placeholder="请输入搜索内容" v-model="input3">
          <el-button type="primary" slot="prepend">任务搜索</el-button>
          <!--<template slot="prepend" style="background-color: #fff;">任务搜索</template>-->
        </el-input>
      </div>
    </div>
    <!---->
    <div class="taskBox" style="box-sizing: border-box;">
      <div class="taskList" v-for="(myTask, index) in myTaskList" :key="index" @click="toDetail(myTask.uid)">
        <div class="taskItem">
          <div><span style="font-size: 16px;color: #409EFF;cursor: pointer;">{{myTask.jobName}}</span></div>
          <div class="taskName"><Icon type="md-ribbon" size="20"/>{{myTask.projectName}}</div>
        </div>
        <div class="taskMsg">
          <div><Icon type="md-contact" size="20"/><span style="padding-left: 8px;">{{myTask.userName}}</span></div>
          <div style="line-height: 20px;">
            <Icon type="md-alarm" size="18"/>
            <span style="padding-left: 8px;float: right;margin-top: 2px;">{{myTask.taskStartDate.split(' ')[0]}} - {{myTask.taskFinishDate.split(' ')[0]}}</span>
          </div>
          <div style="text-align: right;margin-top: 4px;font-size: 12px;" v-if="myTask.dayNum < 0 && myTask.status != '2'">已逾期 <span style="font-size: 18px;color: #f00;font-weight: bold;">{{Math.abs(myTask.dayNum)}}</span> 天</div>
          <div style="text-align: right;margin-top: 4px;font-size: 12px;" v-if="myTask.dayNum >= 0 && myTask.status != '2'">剩余 <span style="font-size: 18px;color: #27CF97;font-weight: bold;">{{myTask.dayNum}}</span> 天</div>
          <div style="text-align: right;margin-top: 4px;font-size: 12px;color: #3a8ee6;font-weight: bold;" v-if="myTask.status === '2'">任务已完成</div>
        </div>
        <div v-bind:class="'taskTag'+ myTask.status">{{myTask.statusStr}}</div>
      </div>
    </div>
    <!---->
    <div style="text-align: center;margin-top: 40px;margin-bottom: 40px;">
      <el-pagination
        background
        layout="prev, pager, next, total"
        @current-change="pageNumChenge($event)"
        :page-size="8"
        :current-page="myTaskViewPayload.pageNum"
        :total="totalRowNum">
      </el-pagination>
    </div>
    <!---->
    <!-- Part05 start 抽屉 任务详情 -->
    <Drawer title="任务详情" class="drawerScroll" :closable="false" width="750" v-model="TaskDetailCompShow">
      <component v-bind:is="compArr.TaskDetailComp"
                 v-bind:taskDrawerOpen="TaskDetailCompShow"
                 v-bind:modifyTaskRes="modifyTaskRes"
                 v-on:FilePreEmit="GetFilePreData"
                 v-on:toPlanDetail="toPlanDetailFuc"
                 v-on:showEditForm="showEditFormFuc"
                 v-on:ActionResThrow="ActionResThrowFuc"
                 v-on:TaskDelCallback="TaskDelCallbackFuc"
                 :nodeId="currentNodeId">
      </component>
    </Drawer>
    <!---->
    <!--修改任务 编辑任务 任务 修改-->
    <Drawer class="drawerScroll" title="修改任务" :closable="false" width="40%" v-model="modifyTaskVisible">
      <!-- 修改任务 编辑任务 引入组件 -->
      <component v-bind:is="compArr.ModifyTask"
                 v-bind:DrawerOpen="modifyTaskVisible"
                 fileFormId="ModifyTask"
                 v-on:FilePreEmit="GetFilePreData"
                 v-on:ModifyTaskCallback="ModifyTaskCallbackFuc"
                 v-on:ShutCompEmit="ShutCompEmitFuc"
                 :nodeId="currentNodeId">
      </component>
    </Drawer>
    <!---->
    <!-- Part06 start 抽屉 计划详情 -->
    <Drawer title="计划详情" class="drawerScroll" :closable="false" width="750" v-model="value444">
      <component v-bind:is="compArr.PlanDetailComp"
                 v-bind:taskDrawerOpen="value444"
                 :nodeId="currentNodeId"
                 v-on:ActionResThrow="ActionResThrowFuc"
                 v-on:addChildMsg="addChildMsgFuc"
                 v-on:toChildMsg="toChildMsgFuc"
                 v-on:PlanDelCallback="PlanDelCallbackFuc">
      </component>
    </Drawer>
    <!--新增 添加计划或者任务 start-->
    <Drawer class="drawerScroll" title="计划表单" :closable="false" width="40%" v-model="bgCoverShow">
      <component v-bind:is="compArr.CreatePlanOrTask"
                 v-bind:DrawerOpen="bgCoverShow"
                 fileFormId="CreatePlanTask"
                 v-on:CreatePlanTaskCallback="CreatePlanTaskCallbackFuc"
                 :nodeId="currentNodeId">
      </component>
    </Drawer>
    <!-- 图片预览 -->
    <el-dialog title="图片预览" :visible.sync="dialogShowImg1">
      <div class="showImg"><img v-bind:src="commentPreviewUrl1" alt=""></div>
    </el-dialog>
  </div>
</template>

<script>
import ModifyTask from './CustomComp/ModifyTask.vue'
import PlanDetailComp from './CustomComp/PlanDetailComp.vue'
import TaskDetailComp from './CustomComp/TaskDetailComp.vue'
import CreatePlanOrTask from './CustomComp/CreatePlanOrTask.vue'
import QuickCreateTaskComp from './CustomComp/QuickCreateTaskComp.vue'
// ModifyTask
export default {
  name: 'MyTaskNew',
  components: {
    ModifyTask,
    PlanDetailComp,
    TaskDetailComp,
    CreatePlanOrTask,
    QuickCreateTaskComp
  },
  data () {
    return {
      compArr: {
        ModifyTask: 'ModifyTask',
        PlanDetailComp: 'PlanDetailComp',
        TaskDetailComp: 'TaskDetailComp',
        CreatePlanOrTask: 'CreatePlanOrTask',
        QuickCreateTaskComp: 'QuickCreateTaskComp'
      },
      // 详情
      modifyTaskRes: '',
      loading3: false,
      taskNameText: '',
      taskForm: {
        jobName: '',
        userName: '',
        jobLevel: 3,
        date1: '',
        date2: '',
        state2: '',
        value9: [],
        value8: [],
        description: '',
        taskUserId: ''
      },
      defImplementer: {
        name: '张三',
        id: ''
      },
      taskRelationShow: false,
      // 所属项目
      projectBelong: '',
      taskIntro: '',
      fileListDis: false,
      fileList: [],
      moreIcon: 'el-icon-arrow-down',
      moreText: '更多',
      options: [],
      selDateStart: '',
      selDateEnd: '',
      // 分类检索
      optionsTask: [{
        value: '2',
        label: '我负责的任务'
      }, {
        value: '3',
        label: '我发起的任务'
      }],
      optionsValue: '2',
      optionsTask2: [{
        value: '',
        label: '全部任务'
      }, {
        value: 'today',
        label: '今日任务'
      }, {
        value: 'overtime',
        label: '逾期任务'
      }, {
        value: 'unstart',
        label: '未开始任务'
      }, {
        value: 'unfinish',
        label: '进行中任务'
      }, {
        value: 'finish',
        label: '已完成任务'
      }],
      optionsValue2: '',
      optionsValue3: '',
      projectList: [],
      myTaskViewPayload: {
        pageNum: 1,
        pageSize: '8',
        projectName: '',
        jobName: '',
        taskSource: '2',
        sType: ''
      },
      // 默认指派
      input3: '',
      myTaskList: [],
      totalRowNum: 0,
      // 详情
      TaskDetailCompShow: false,
      currentNodeId: '',
      // 修改任务
      modifyTaskVisible: false,
      // 预览
      commentPreviewUrl1: '',
      // 预览
      dialogShowImg1: false,
      // 计划详情
      value444: false,
      // addNode 表单
      activeNameBgCover: 'first',
      bgCoverShow: false
    }
  },
  created () {
    this.queryMyTaskView()
    this.queryMyProjectList()
  },
  watch: {
    // optionsValue3: function (newValue, oldValue) {
    //   this.myTaskViewPayload.projectName = newValue
    //   this.queryMyTaskView()
    // }
    input3: function (newValue, oldValue) {
      this.myTaskViewPayload.jobName = newValue
      this.myTaskViewPayload.pageNum = 1
      this.queryMyTaskView()
    }
  },
  methods: {
    queryMyProjectList () {
      var that = this
      this.ajax('/myProject/myProjectList', {}).then(res => {
        // this.log('选择所属项目:', res)
        if (res.code === 200) {
          that.projectList = res.data
        }
      })
    },
    myTaskStyleChange: function (e) {
      // console.log('eeeee', e)
      var that = this
      that.myTaskViewPayload.pageNum = 1
      that.myTaskViewPayload.taskSource = e
      that.queryMyTaskView()
    },
    taskTypeState: function (e) {
      var that = this
      that.myTaskViewPayload.pageNum = 1
      that.myTaskViewPayload.sType = e
      that.queryMyTaskView()
    },
    taskOfProject: function (e) {
      var that = this
      that.myTaskViewPayload.pageNum = 1
      that.myTaskViewPayload.projectName = e
      that.queryMyTaskView()
    },
    queryMyTaskView () {
      var that = this
      this.ajax('/myTask/myTaskView', that.myTaskViewPayload).then(res => {
        if (res.code === 200) {
          // that.log('myTaskView:', res)
          that.myTaskList = res.data.list
          that.totalRowNum = res.data.totalRow
        }
      })
    },
    pageNumChenge (e) {
      // console.log('eeeeeeee', e)
      this.myTaskViewPayload.pageNum = e
      this.queryMyTaskView()
    },
    toDetail: function (id) {
      this.currentNodeId = id
      this.TaskDetailCompShow = true
    },
    // 任务详情组件
    // 附件 附件预览
    GetFilePreData (obj) {
      if (obj.previewUrl && this.isImage(obj.fileName)) {
        this.showBigImage1(obj.previewUrl)
      }
    },
    showBigImage1: function (url) {
      if (url) {
        this.commentPreviewUrl1 = url
        this.dialogShowImg1 = true
      }
    },
    toPlanDetailFuc: function (id) {
      var that = this
      that.currentNodeId = id
      that.value444 = true
      that.TaskDetailCompShow = false
    },
    showEditFormFuc: function () {
      var that = this
      that.modifyTaskVisible = true
    },
    ActionResThrowFuc: function (obj) {
      var that = this
      if (obj.res.code === 200) {
        switch (obj.actionName) {
          case 'transferTask':
            // 任务移交
            that.TaskDetailCompShow = false
            that.queryMyTaskView(that.activeId)
            break
          case 'finishTask':
            // 任务完成
            that.queryMyTaskView()
            break
          case 'restartTask':
            // 任务重启
            that.queryMyTaskView()
            break
          case 'startTask':
            // 任务开始
            that.queryMyTaskView()
            break
          case 'decomposeTask':
            // 任务分解
            that.queryMyTaskView()
            break
          default:
            this.log('')
        }
      }
    },
    TaskDelCallbackFuc: function (res) {
      var that = this
      if (res.code === 200) {
        that.queryMyTaskView()
        that.TaskDetailCompShow = false
        that.$message({
          message: '删除成功！',
          type: 'success'
        })
      } else {
        that.$message({
          message: res.msg,
          type: 'warning'
        })
      }
    },
    // 编辑任务 修改任务
    ModifyTaskCallbackFuc: function (res) {
      var that = this
      if (res.code === 200) {
        that.modifyTaskVisible = false
        that.queryMyTaskView()
        that.$message({
          message: '修改成功！',
          type: 'success'
        })
      } else {
        that.$message({
          message: res.msg,
          type: 'warning'
        })
      }
    },
    // 组件内点击了关闭 父组件执行关闭子组件操作
    ShutCompEmitFuc: function (res) {
      this[res] = false
    },
    // 计划 计划删除 返回结果处理
    PlanDelCallbackFuc: function (res) {
      var that = this
      if (res.code === 200) {
        that.queryMyTaskView()
        that.value444 = false
        that.$message({
          message: '删除成功！',
          type: 'success'
        })
      } else {
        that.$message({
          message: res.msg,
          type: 'warning'
        })
      }
    },
    toChildMsgFuc: function (id) {
      var that = this
      that.currentNodeId = id
      that.value444 = false
      that.TaskDetailCompShow = true
    },
    addChildMsgFuc: function (data) {
      this.addNode(data.id, data.type)
    },
    // 新建 添加子节点
    addNode: function (nodeId, nodeType) {
      var that = this
      if (nodeType) {
        if (nodeType === '1' || nodeType === '计划') {
          that.activeNameBgCover = 'first'
        } else {
          that.activeNameBgCover = 'second'
        }
      }
      this.currentNodeId = nodeId
      this.bgCoverShow = true
    },
    // 新建计划 新建任务 引入组件 返回
    CreatePlanTaskCallbackFuc: function (res) {
      var that = this
      if (res === 'shut') {
        that.bgCoverShow = false
      } else {
        if (res.code === 200) {
          that.bgCoverShow = false
          that.queryMyTaskView()
          that.$message({
            message: '创建成功！',
            type: 'success'
          })
        } else {
          that.$message({
            message: res.msg,
            type: 'warning'
          })
        }
      }
    }
  }
}
</script>

<style scoped>
  .MyTaskNew{
    position: relative;
  }
  .searchItem{
    margin-top: 10px;
    display: flex;
    padding: 10px;
    background-color: #f5f5f8;
  }
  .searchItem>div{
    margin-right: 10px;
  }
  .el-input-group__prepend{
    background-color: #fff;
  }
  .taskList{
    height: 100px;
    width: 100%;
    display: flex;
    padding: 20px;
    justify-content: space-between;
    border-bottom: 1px dashed #ddd;
    position: relative;
    overflow: hidden;
  }
  .taskList:hover,.taskList2:hover{
    background-color: #f5f8fa;
  }
  .taskList:hover .taskItem{
    height: 100%;
    width: 50%;
    border-left: 6px solid #409EFF;
    /*display: flex;*/
    padding-left: 20px;
  }
  .taskItem{
    height: 100%;
    width: 50%;
    border-left: 6px solid #ccc;
    /*display: flex;*/
    padding-left: 20px;
  }
  .taskName{
    padding: 10px;
    font-size: 14px;
    color: #888;
  }
  .taskMsg{
    color: #888;
    width: 50%;
    text-align: right;
    padding-right: 40px;
    font-size: 14px;
  }
  /*任务状态 标*/
  .taskTag0{
    height: 20px;
    text-align: center;
    width: 100px;
    font-size: 12px;
    background-color: #ffc107;
    transform: rotate(45deg);
    position: absolute;
    top: 8px;
    right: -32px;
    color: #fff;
  }
  .taskTag1{
    height: 20px;
    text-align: center;
    width: 100px;
    font-size: 12px;
    background-color: #27CF97;
    transform: rotate(45deg);
    position: absolute;
    top: 8px;
    right: -32px;
    color: #fff;
  }
  .taskTag2{
    height: 20px;
    text-align: center;
    width: 100px;
    font-size: 12px;
    background-color: #3a8ee6;
    transform: rotate(45deg);
    position: absolute;
    top: 8px;
    right: -32px;
    color: #fff;
  }
  .taskTag4{
    height: 20px;
    text-align: center;
    width: 100px;
    font-size: 12px;
    background-color: #ccc;
    transform: rotate(45deg);
    position: absolute;
    top: 8px;
    right: -32px;
    color: #fff;
  }
</style>
