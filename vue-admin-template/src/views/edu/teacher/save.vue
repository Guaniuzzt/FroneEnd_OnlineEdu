<template>
  <div class="app-container">
    Add Teacher
    <el-form label-width="120px">
      <el-form-item label="Teacher Name">
        <el-input v-model="teacher.name"/>
      </el-form-item>
      <el-form-item label="Sort">
        <el-input-number v-model="teacher.sort" controls-position="right" min="0"/>
      </el-form-item>
      <el-form-item label="Teacher Title">
        <el-select v-model="teacher.level" clearable placeholder="Select">
          <!--
            数据类型一定要和取出的json中的一致，否则没法回填
            因此，这里value使用动态绑定的值，保证其数据类型是number
          -->
          <el-option :value="1" label="Senior Teacher"/>
          <el-option :value="2" label="Chief Teacher"/>
        </el-select>
      </el-form-item>
      <el-form-item label="Experience">
        <el-input v-model="teacher.career"/>
      </el-form-item>
      <el-form-item label="Teacher Introduction">
        <el-input v-model="teacher.intro" :rows="10" type="textarea"/>
      </el-form-item>

      <!-- 讲师头像：TODO -->

      <el-form-item>
        <el-button :disabled="saveBtnDisabled" type="primary" @click="saveOrUpdate">Save</el-button>
      </el-form-item>
    </el-form>
  </div>
</template>

<script>
import teacherApi from '@/api/edu/teacher'
export default {
    data(){
      return{
        teacher:{ 
          // name: '',
          // sort: ,
          // level: 1,
          // career: '',
          // intro: '',
          // avatar: ''
        },
        saveBtnDisabled: false // 保存按钮是否禁用,
      }
    },
    created(){
      console.log('created')
      this.init()
    
    },
    watch:{
      $route(to, from) {   //当路由发生变化，方法就会执行
        console.log('watch $route')
        this.init()
      }
    },
    methods:{

      init(){
         //如果路径中有id值
        if (this.$route.params && this.$route.params.id) {
          const id = this.$route.params.id
          this.getInfo(id)
        }else{
          this.teacher={}
        }

      },
      //根据讲师ID查询的方法
      getInfo(id){
        teacherApi.getTeacherInfo(id)
        .then(response =>{
          this.teacher = response.data.teacher
        })
        .catch((response) => {
          this.$message({
            type: 'error',
            message: '获取数据失败'
          })
        })
      },

      saveOrUpdate() {
        //判断修改还是添加
        //根据teacher是否有id
        if(!this.teacher.id){
          this.saveTeacher()
        }else{
          this.updateTeacher()
        }
        
      },
      //修改讲师的方法
      updateTeacher(){
        teacherApi.updateTeacherInfo(this.teacher)
          .then(response =>{
            this.$message({
                type: 'success',
                message: 'Update completed'
            });
            //返回列表   路由跳转
            this.$router.push({path:'/teacher/table'})
          })
      },


      // 保存
      saveTeacher() {
        teacherApi.addTeacher(this.teacher)
        .then(response =>{  //add sucess
            //提示信息
            this.$message({
                type: 'success',
                message: 'Add completed'
            });
            //返回列表   路由跳转
            this.$router.push({path:'/teacher/table'})
        })
      }
    }
}
</script>