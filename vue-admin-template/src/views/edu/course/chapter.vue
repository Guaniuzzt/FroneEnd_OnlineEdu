<template>

  <div class="app-container">

    <h2 style="text-align: center;">Pulish A New Course</h2>

    <el-steps :active="2" process-status="wait" align-center style="margin-bottom: 40px;">
      <el-step title="Enter Course Information"/>
      <el-step title="Create Course Outline"/>
      <el-step title="Final Verison"/>
    </el-steps>

    <el-button type="text" @click="openChapterDialog()">Add Chapter</el-button>

    <ul class="chanpterList">
      <li
          v-for="chapter in chapterVideoList"
          :key="chapter.id">
          <p>
              {{ chapter.title }} 
              <span class="acts">
                <el-button style="" type="text" @click="openVideo(chapter.id)">Add Section</el-button>
                <el-button style="" type="text" @click="openEditChapter(chapter.id)">Edit</el-button>
                <el-button type="text"  @click="removeChapter(chapter.id)">Delete</el-button>
              </span>
          </p>

          <!-- 视频 -->
          <ul class="chanpterList videoList">
              <li
                  v-for="video in chapter.children"
                  :key="video.id">
                  <p>
                    {{ video.title }}
                    <span class="acts">
                        <el-button type="text" @click="openEditVideo(video.id)">Edit</el-button>
                        <el-button type="text" @click="removeVideo(video.id)">Delete</el-button>
                    </span>
                  </p>
              </li>
          </ul>
        </li>
      </ul>


    <div>
        <el-button @click="previous">Back</el-button>
        <el-button :disabled="saveBtnDisabled" type="primary" @click="next">Next</el-button>
    </div>

    <!-- 添加和修改章节表单 -->
    <el-dialog :visible.sync="dialogChapterFormVisible" title="添加章节">
        <el-form :model="chapter" label-width="120px">
            <el-form-item label="Chapter Title">
                <el-input v-model="chapter.title"/>
            </el-form-item>
            <el-form-item label="Chapter Sort">
                <el-input-number v-model="chapter.sort" :min="0" controls-position="right"/>
            </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer">
            <el-button @click="dialogChapterFormVisible = false">Cancel</el-button>
            <el-button type="primary" @click="saveOrUpdate">Confirm</el-button>
        </div>
    </el-dialog>

    <!-- 添加和修改课时表单 -->
    <el-dialog :visible.sync="dialogVideoFormVisible" title="添加课时">
      <el-form :model="video" label-width="120px">
        <el-form-item label="课时标题">
          <el-input v-model="video.title"/>
        </el-form-item>
        <el-form-item label="课时排序">
          <el-input-number v-model="video.sort" :min="0" controls-position="right"/>
        </el-form-item>
        <el-form-item label="是否免费">
          <el-radio-group v-model="video.isFree">
            <el-radio :label=1>免费</el-radio>
            <el-radio :label=0>默认</el-radio>
          </el-radio-group>
        </el-form-item>
        <el-form-item label="上传视频">
          <!-- TODO -->
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogVideoFormVisible = false">取 消</el-button>
        <el-button :disabled="saveVideoBtnDisabled" type="primary" @click="saveOrUpdateVideo">确 定</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import chapter from '@/api/edu/chapter'
import video from '@/api/edu/video'
export default {
  data() {
    return {
      saveBtnDisabled: false, // 保存按钮是否禁用
      chapterVideoList: [],
      courseId: '',
      chapter:{ //封装章节数据
        title: '', 
        sort: 0
      },
      video:{
        title: '',
        sort: 0,
        isFree: 0,
        videoSourceId: ''
      },
      dialogChapterFormVisible: false, //章节弹框的值
      dialogVideoFormVisible: false,   //小节弹框的值
    }
  },

  created() {
    console.log('chapter created')
    //获取路由里的id值
    if(this.$route.params && this.$route.params.id){
      this.courseId = this.$route.params.id
      this.getChapterVideo()
    }
    
  },

  methods: {


  //=================================小节操作======================================
    //删除小节
    removeVideo(videoid){
        this.$confirm('This will permanently delete the record. Continue?', 'Warning', {
                confirmButtonText: 'OK',
                cancelButtonText: 'Cancel',
                type: 'warning'
            }).then(() => {   //click ok
              //删除方法
              video.deleteVideo(videoid)
              .then(response =>{
                this.$message({
                  type: 'success',
                  message: 'Delete completed'
                });
                //刷新页面
                this.getChapterVideo()
              }) 
            }).catch(() => {
              //   this.$message({
              //   type: 'info',
              //   message: 'Delete canceled'
              // }); 
        }); 

    },
    
    //修改小节弹窗
    openEditVideo(videoid){
      this.dialogVideoFormVisible = true
      video.getVideo(videoid)   //回显
            .then(response =>{
              this.video = response.data.video
            })
    },
    //添加小节弹窗的方法
    openVideo(chapterId){
        this.dialogVideoFormVisible = true

        this.video = {
          title: '',
          sort: 0,
          isFree: 0,
          videoSourceId: ''
        }

        //设置章节id
        this.video.chapterId = chapterId
        //设置课程id
        this.video.courseId = this.courseId

    },

    //添加小节
    addVideo(){
        video.addVideo(this.video)
            .then(respnse =>{
                 //关闭弹框
                this.dialogVideoFormVisible = false
                //提示
                this.$message({
                    type: 'success',
                    message: '添加小节成功!'
                });
                //刷新页面
                this.getChapterVideo()
            })
    },

    //更新小节
    updateVideo(){
      video.updateVideo(this.video)
            .then(response =>{
                //关闭弹框
                this.dialogVideoFormVisible = false
                //提示
                this.$message({
                    type: 'success',
                    message: '修改小节成功!'
                });
                //刷新页面
                this.getChapterVideo()
            })
    },

    saveOrUpdateVideo(){
        if(!this.video.id){
          this.addVideo()
        }else{
          this.updateVideo()
        }
    },
    





    //=====================================章节操作===============================

    //修改章节弹框数据回显
    openEditChapter(chapterId){
        this.dialogChapterFormVisible = true  //弹框
        chapter.getChapter(chapterId)     //数据回显
              .then(response =>{
                  this.chapter = response.data.chapter
              })
    },

    //删除章节
    removeChapter(chapterId){
        this.$confirm('This will permanently delete the record. Continue?', 'Warning', {
                confirmButtonText: 'OK',
                cancelButtonText: 'Cancel',
                type: 'warning'
            }).then(() => {   //click ok
              //删除方法
              chapter.deleteChapter(chapterId)
              .then(response =>{
                this.$message({
                  type: 'success',
                  message: 'Delete completed'
                });
                //刷新页面
                this.getChapterVideo()
              }) 
            }).catch(() => {
              //   this.$message({
              //   type: 'info',
              //   message: 'Delete canceled'
              // }); 
        }); 
    },

    
    //弹出添加章节页面
    openChapterDialog() {
        //弹框
        this.dialogChapterFormVisible = true
        //表单数据清空
        this.chapter = { //清空
          title: '', 
          sort: 0
      }
    },
    //添加章节
    addChapter() {
        //设置课程id到chapter对象里面
        this.chapter.courseId = this.courseId
        chapter.addChapter(this.chapter)
            .then(response => {
                //关闭弹框
                this.dialogChapterFormVisible = false
                //提示
                this.$message({
                    type: 'success',
                    message: '添加章节成功!'
                });
                //刷新页面
                this.getChapterVideo()
            })
    },

    //修改章节方法
    updateChapter(){
        chapter.updateChapter(this.chapter)
                .then(response => {
                    //关闭弹框
                    this.dialogChapterFormVisible = false
                    //提示
                    this.$message({
                        type: 'success',
                        message: '修改章节成功!'
                    });
                    //刷新页面
                    this.getChapterVideo()
                })
    },
    
    saveOrUpdate(){
      if(!this.chapter.id ){
        this.addChapter()
      }else{
        this.updateChapter()
      }
      
    },
    //根据课程id查询章节和小节的内容
    getChapterVideo(){
      chapter.getAllChapterVideo(this.courseId)
      .then(response =>{
        this.chapterVideoList = response.data.allChapterVideo
      })
    },
    previous() {
      console.log('previous')
      this.$router.push({ path: '/course/info/'  + this.courseId})
    },

    next() {
      console.log('next')
      this.$router.push({ path: '/course/publish/' + this.courseId})
    }
  }
}
</script>


<style scoped>
.chanpterList{
    position: relative;
    list-style: none;
    margin: 0;
    padding: 0;
}
.chanpterList li{
  position: relative;
}
.chanpterList p{
  float: left;
  font-size: 20px;
  margin: 10px 0;
  padding: 10px;
  height: 70px;
  line-height: 50px;
  width: 100%;
  border: 1px solid #DDD;
}
.chanpterList .acts {
    float: right;
    font-size: 14px;
}

.videoList{
  padding-left: 50px;
}
.videoList p{
  float: left;
  font-size: 14px;
  margin: 10px 0;
  padding: 10px;
  height: 50px;
  line-height: 30px;
  width: 100%;
  border: 1px dotted #DDD;
}

</style>