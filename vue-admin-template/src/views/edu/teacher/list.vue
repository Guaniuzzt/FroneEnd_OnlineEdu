<template>
  <div class="app-container">
    Teacher List
    <!--查询表单-->
    <el-form :inline="true" class="demo-form-inline">
      <el-form-item>
        <el-input v-model="teacherQuery.name" placeholder="Teacher's Name"/>
      </el-form-item>

      <el-form-item>
        <el-select v-model="teacherQuery.level" clearable placeholder="Teacher's title">
          <el-option :value="1" label="Senior Teacher"/>
          <el-option :value="2" label="Chief Teacher"/>
        </el-select>
      </el-form-item>

      <el-form-item label="Add Date">
        <el-date-picker
          v-model="teacherQuery.begin"
          type="datetime"
          placeholder="Start Date"
          value-format="yyyy-MM-dd HH:mm:ss"
          default-time="00:00:00"
        />
      </el-form-item>
      <el-form-item>
        <el-date-picker
          v-model="teacherQuery.end"
          type="datetime"
          placeholder="End Date"
          value-format="yyyy-MM-dd HH:mm:ss"
          default-time="00:00:00"
        />
      </el-form-item>

      <el-button type="primary" icon="el-icon-search" @click="getList()">Search</el-button>
      <el-button type="default" @click="resetData()">Clear</el-button>
    </el-form>
        <!-- 表格 -->
        <!-- : 就是v-bind 单向绑定 -->
        <!-- v-loading="listLoading" -->
    <el-table
      :data="list"             
      element-loading-text='List Loading'
      border
      fit
      highlight-current-row>

      <el-table-column
        label="No."
        width="70"
        align="center">
        <template slot-scope="scope">
          {{ (page - 1) * limit + scope.$index + 1 }}
        </template>
      </el-table-column>

      <el-table-column prop="name" label="Name" width="80" />

      <el-table-column label="Title" width="80">
        <template slot-scope="scope">
          {{ scope.row.level===1?'高级讲师':'首席讲师' }}
        </template>
      </el-table-column>

      <el-table-column prop="intro" label="Experience" />

      <el-table-column prop="gmtCreate" label="Add Time" width="160"/>

      <el-table-column prop="sort" label="Sort" width="60" />

      <el-table-column label="Setting" width="200" align="center">
        <template slot-scope="scope">
          <router-link :to="'/teacher/edit/'+scope.row.id">
            <el-button type="primary" size="mini" icon="el-icon-edit">Edit</el-button>
          </router-link>
          <el-button type="danger" size="mini" icon="el-icon-delete" @click="removeDataById(scope.row.id)">Delete</el-button>
        </template>
      </el-table-column>
    </el-table>

       <!-- 分页 -->
    <el-pagination
      :current-page="page"
      :page-size="limit"
      :total="total"
      style="padding: 30px 0; text-align: center;"
      layout="total, prev, pager, next, jumper"
      @current-change="getList"
    />
  </div>
</template>
//引入调用teacher.js


<script>
import teacher from '@/api/edu/teacher'

export default {
    //核心代码位置
    data(){  //定义变量初始值
        return{
            list: null,
            page:1,
            limit:5,
            total:0,
            teacherQuery:{} 
        }
    },
    created(){
        this.getList()
    },
    methods:{  //创建具体的方法，调用teacher.js定义的方法
        //讲师列表的方法
        getList(page = 1){
            this.page = page
            teacher.getTeacherListPage(this.page, this.limit, this.teacherQuery)
            .then(reponse =>{ //request success
                this.list = reponse.data.rows
                this.total = reponse.data.total
                console.log(this.list)
                console.log(this.total)
            }) 
            .catch(error =>{   //request error
                console.log(error)
            }) 
        },
        resetData(){
            //清空表达
            this.teacherQuery = {}
            //查询
            this.getList()
        },
        removeDataById(id){
            this.$confirm('This will permanently delete the record. Continue?', 'Warning', {
                confirmButtonText: 'OK',
                cancelButtonText: 'Cancel',
                type: 'warning'
            }).then(() => {   //click ok
              //删除方法
              teacher.deleteTeacherId(id)
              .then(response =>{
                this.$message({
                  type: 'success',
                  message: 'Delete completed'
                });
                this.getList()
              }) 
            }).catch(() => {
              //   this.$message({
              //   type: 'info',
              //   message: 'Delete canceled'
              // }); 
          });      
        }
    }

}
</script>