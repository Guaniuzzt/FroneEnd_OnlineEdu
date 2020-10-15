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
      <el-form-item label="讲师头像">

          <!-- 头衔缩略图 -->
          <pan-thumb :image="teacher.avatar"/>
          <!-- 文件上传按钮 -->
          <el-button type="primary" icon="el-icon-upload" @click="imagecropperShow=true">Upload Avatar
          </el-button>

          <!--
          v-show：是否显示上传组件
          :key：类似于id，如果一个页面多个图片上传控件，可以做区分
          :url：后台上传的url地址
          @close：关闭上传组件
          @crop-upload-success：上传成功后的回调 -->
          <image-cropper
            v-show="imagecropperShow"
            :width="300"
            :height="300"
            :key="imagecropperKey"
            :url="BASE_API+'/eduoss/fileoss'"
            field="file"
            @close="close"
            @crop-upload-success="cropSuccess"/>
      </el-form-item>

      <el-form-item>
        <el-button :disabled="saveBtnDisabled" type="primary" @click="saveOrUpdate">Save</el-button>
      </el-form-item>
    </el-form>
  </div>
</template>

<script>
import teacherApi from '@/api/edu/teacher'
import ImageCropper from '@/components/ImageCropper'
import PanThumb from '@/components/PanThumb'
export default {
    components:{ImageCropper, PanThumb},
    data(){
      return{
        teacher:{ 
           name: '',
           sort: 0,
           level: 1,
           career: '',
           intro: '',
           avatar: ''
        },
        //上传弹框的组件是否显示
        imagecropperShow: false,
        imagecropperKey: 0,
        BASE_API: process.env.BASE_API, //获取dev.env.js里面端口号
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

      close(){  //关闭上传弹框的方法
        this.imagecropperShow = false
        //上传组件初始化
        this.imagecropperKey = this.imagecropperKey + 1
      },
      //上传成功方法
      cropSuccess(data){
        this.imagecropperShow = false
        this.imagecropperKey = this.imagecropperKey + 1
        //上传之后返回图片地址
        this.teacher.avatar = data.url
      },

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