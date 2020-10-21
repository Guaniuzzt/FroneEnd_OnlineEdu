<template>

  <div class="app-container">

    <h2 style="text-align: center;">Publish A New Course</h2>

    <el-steps :active="1" process-status="wait" align-center style="margin-bottom: 40px;">
      <el-step title="Enter Course Information"/>
      <el-step title="Create Course Outline"/>
      <el-step title="Final Verison"/>
    </el-steps>

    <el-form label-width="150px">

        <el-form-item label="Course Title">
            <el-input v-model="courseInfo.title" placeholder=" Example: Machine Learning"/>
        </el-form-item>

        <!-- 所属分类 TODO -->
         <el-form-item label="Course Category">
            <el-select
                v-model="courseInfo.subjectParentId"
                placeholder="Please Choose"  @change="subjectLevelOneChanged">
                <el-option
                    v-for=" subject in subjectOneList"
                    :key="subject.id"
                    :label="subject.title"
                    :value="subject.id"/>
            </el-select>

            <el-select v-model="courseInfo.subjectId" placeholder="Please Choose">
                <el-option
                    v-for="subject in subjectTwoList"
                    :key="subject.id"
                    :label="subject.title"
                    :value="subject.id"/>
            </el-select>
        </el-form-item>

        <!-- 课程讲师 TODO -->
        <!-- 课程讲师 -->
        <el-form-item label="Teacher">
            <el-select
                v-model="courseInfo.teacherId"
                placeholder="Please Choose">
                <el-option
                    v-for="teacher in teacherList"
                    :key="teacher.id"
                    :label="teacher.name"
                    :value="teacher.id"/>
            </el-select>
        </el-form-item>

        <el-form-item label="Course Hour">
            <el-input-number :min="0" v-model="courseInfo.lessonNum" controls-position="right" placeholder="请填写课程的总课时数"/>
        </el-form-item>

            <!-- 课程简介 TODO -->
        <!-- 课程简介-->
        <el-form-item label="Course Description">
            <tinymce :height="300" v-model="courseInfo.description"/>
        </el-form-item>

            <!-- 课程封面 TODO -->
        <el-form-item label="Course Cover">
            <el-upload
                :show-file-list="false"
                :on-success="handleAvatarSuccess"
                :before-upload="beforeAvatarUpload"
                :action="BASE_API+'/eduoss/fileoss'"
                class="avatar-uploader">
                <img :src="courseInfo.cover">
            </el-upload>
        </el-form-item>

        <el-form-item label="Course Price">
            <el-input-number :min="0" v-model="courseInfo.price" controls-position="right" placeholder="免费课程请设置为0元"/> Yuan
        </el-form-item>

        <el-form-item>
            <el-button :disabled="saveBtnDisabled" type="primary" @click="saveOrUpdate">Save And Next</el-button>
        </el-form-item>
    </el-form>
  </div>
</template>

<script>
import course from '@/api/edu/course'
import subject from '@/api/edu/subject'
import Tinymce from '@/components/Tinymce'  //引入组件

export default {

  //声明组件
  components:{Tinymce},
  data() {
    return {
      saveBtnDisabled: false, // 保存按钮是否禁用
      courseInfo:{
            title: '',
            subjectId: '',  //二级分类id
            subjectParentId: '', //一级分类id
            teacherId: '',
            lessonNum: 0,
            description: '',
            cover: '/static/testpicture.jpg',
            price: 0
      },
      cousreid : '',

      teacherList:[] , //封装所有的讲师数据

      subjectOneList: [], //一级分类

      subjectTwoList: [],  // 二级分类

      BASE_API: process.env.BASE_API // 接口API地址
    }
  },

  created() {

    //获取路由中的id值
    if(this.$route.params && this.$route.params.id){
      this.courseId = this.$route.params.id
      this.getTeacher()
      this.getInfo()
      
    }else{
        //console.log('info created')
        //this.getTeacher()
        //初始化一级分类
        this.getTeacher()
        this.getOneSubject()
        this.courseInfo = {
            title: '',
            subjectId: '',  //二级分类id
            subjectParentId: '', //一级分类id
            teacherId: '',
            lessonNum: 0,
            description: '',
            cover: '/static/testpicture.jpg',
            price: 0
        }
    }
  },

  methods: {
    getInfo(){
        course.getCourseInfoId(this.courseId)
            .then(response =>{
                //再courseInfo中有课程基本信息，包含一级分类id 和 二级分类id
                this.courseInfo = response.data.courseInfoVo
                //1 查询所有分类，包含一级和二级
                subject.getSubjectList()
                    .then(response =>{
                        //2 获取所有一级分类
                        this.subjectOneList = response.data.list

                        //3 把所有的一级分类数组进行遍历比较
                        for(var i=0; i<this.subjectOneList.length; i++){
                            var oneSubject = this.subjectOneList[i]
                            if(this.courseInfo.subjectParentId == oneSubject.id){
                                //获取一级分类的所有的二级分类
                                this.subjectTwoList = oneSubject.children
                            }
                        }
                    })
            })
    },
    beforeAvatarUpload(file){ //上传封面之前
        const isJPG = file.type === 'image/jpeg'
        const isLt2M = file.size / 1024 / 1024 < 2

        if (!isJPG) {
            this.$message.error('Only .JPG Format!')
        }
        if (!isLt2M) {
            this.$message.error('File Size Limit 2MB!')
        }
        return isJPG && isLt2M

    },
    handleAvatarSuccess(res, file){ //上传封面成功
        console.log(res)// 上传响应
        console.log(URL.createObjectURL(file.raw))// base64编码
        this.courseInfo.cover = res.data.url
    },

    //点击某个一级分类触发change，显示对应二级分类
    subjectLevelOneChanged(value){ //value就是一级分类id值
        //alert(value)
        for(var i=0; i<this.subjectOneList.length; i++){
            var oneSubject = this.subjectOneList[i]
            //判断一级分类id和点击的分类id是否相同
            if(value == oneSubject.id){
                this.subjectTwoList = oneSubject.children
                this.courseInfo.subjectId = ''
            }
        }


    },
    //查询所有的一级查询
    getOneSubject(){
        subject.getSubjectList()
            .then(response =>{
                this.subjectOneList = response.data.list
            })
    },

    //查询所有的讲师
    getTeacher(){
        course.getListTeacher()
            .then(response =>{
                this.teacherList = response.data.items
            })
    },

    addCourse(){
      console.log('next')
      course.addCourseInfo(this.courseInfo)
        .then( response =>{
            this.$message({
            type: 'success',
            message: 'Sucessfully add course information'
            })
            this.$router.push({ path: '/course/chapter/'+response.data.courseId})
        })
      
    },

    updateCourse(){
        course.UpdateCourseInfoId(this.courseInfo)
            .then(response =>{
                this.$message({
                type: 'success',
                message: 'Sucessfully edit course information'
                })
                this.$router.push({ path: '/course/chapter/'+this.courseId})
            })
    },
    saveOrUpdate(){
        //判断添加还是修改
        if(!this.courseInfo.id){
            this.addCourse()
        }else{
            this.updateCourse()
        }
    }
  }
}
</script>

<style scoped>
.tinymce-container {
  line-height: 29px;
}
</style>