<template>

  <div class="app-container">

    <el-button type="text" @click="dialogChapterFormVisible = true">添加章节</el-button>

    <!-- 添加和修改章节表单 -->
    <el-dialog :visible.sync="dialogChapterFormVisible" title="添加章节">
        <el-form :model="chapter" label-width="120px">
            <el-form-item label="章节标题">
                <el-input v-model="chapter.title"/>
            </el-form-item>
            <el-form-item label="章节排序">
                <el-input-number v-model="chapter.sort" :min="0" controls-position="right"/>
            </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer">
            <el-button @click="dialogChapterFormVisible = false">取 消</el-button>
            <el-button type="primary" @click="saveOrUpdate">确 定</el-button>
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
            <el-radio-group v-model="video.free">
              <el-radio :label="true">免费</el-radio>
              <el-radio :label="false">默认</el-radio>
            </el-radio-group>
          </el-form-item>
        <el-form-item label="上传视频">
            <el-upload
                  :on-success="handleVodUploadSuccess"
                  :on-remove="handleVodRemove"
                  :before-remove="beforeVodRemove"
                  :on-exceed="handleUploadExceed"
                  :file-list="fileList"
                  :action="BASE_API+'vodservice/upload'"
                  :limit="1"
                  class="upload-demo">
            <el-button size="small" type="primary">上传视频</el-button>
            <el-tooltip placement="right-end">
                <div slot="content">最大支持1G，<br>
                    支持3GP、ASF、AVI、DAT、DV、FLV、F4V、<br>
                    GIF、M2T、M4V、MJ2、MJPEG、MKV、MOV、MP4、<br>
                    MPE、MPG、MPEG、MTS、OGG、QT、RM、RMVB、<br>
                    SWF、TS、VOB、WMV、WEBM 等视频格式上传</div>
                <i class="el-icon-question"/>
            </el-tooltip>
            </el-upload>
        </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer">
          <el-button @click="dialogVideoFormVisible = false">取 消</el-button>
          <el-button :disabled="saveVideoBtnDisabled" type="primary" @click="saveOrUpdateVideo">确 定</el-button>
        </div>
      </el-dialog>
<!-- 章节 -->
<ul class="chanpterList">
    <li
        v-for="chapter in chapterNestedList"
        :key="chapter.id">
        <p>
            {{ chapter.title }}

            <span class="acts">
                <el-button type="text" @click="dialogVideoFormVisible = true; chapterId = chapter.id">添加课时</el-button>
                <el-button type="text" @click="editChapter(chapter.id)">编辑</el-button>
                <el-button type="text" @click="removeChapter(chapter.id)">删除</el-button>
            </span>
        </p>

        <!-- 视频 -->
        <ul class="chanpterList videoList">
            <li
                v-for="video in chapter.children"
                :key="video.id">
                <p>{{ video.title }}
                    <span class="acts">
                        <el-button type="text" @click="editVideo(video.id)">编辑</el-button>
                        <el-button type="text" @click="removeVideo(video.id)">删除</el-button>
                    </span>
                </p>
            </li>
        </ul>
    </li>
</ul>
<div>
    <el-button @click="previous">上一步</el-button>
    <el-button :disabled="saveBtnDisabled" type="primary" @click="next">下一步</el-button>
</div>
  </div>
</template>
<script>
import chapter from "@/api/edu/chapter"
import video from "@/api/edu/video"
import vod from '@/api/edu/vod'
export default {
  data() {
    return {
      saveBtnDisabled: false, // 保存按钮是否禁用
      courseId: '', // 所属课程id
      chapterNestedList: [], // 章节嵌套课时列表、
      dialogChapterFormVisible: false, //是否显示章节表单
      chapter: {// 章节对象
        id:'',
        title: '',
        sort: 0
      },
      saveVideoBtnDisabled: false, // 课时按钮是否禁用
      dialogVideoFormVisible: false, // 是否显示课时表单
      chapterId: '', // 课时所在的章节id
      video: {// 课时对象
            title: '',
            sort: 0,
            free: false,
            videoSourceId: '',
            videoOriginalName: ''
          },
      fileList: [],//上传文件列表
      BASE_API: process.env.BASE_API // 接口API地址
    }
  },

  created() {
    console.log('info created')
    this.init()
  },

  methods: {
    init(){
        if (this.$route.params && this.$route.params.id) {
        this.courseId =  this.$route.params.id
        // 根据id获取课程基本信息
        this.fetchChapterNestedListByCourseId()
        }
    },
// 根据id获取课程基本信息
    fetchChapterNestedListByCourseId(){
      chapter.getNestedTreeList(this.courseId).then(resopnse =>{
        this.chapterNestedList = resopnse.data.items
      })

    },

    saveOrUpdateVideo() {
      this.saveVideoBtnDisabled = true
      if (!this.video.id) {
        this.saveDataVideo()
      } else {
        this.updateDataVideo()
      }
    },

    //新增小节
    saveDataVideo() {
      this.video.courseId = this.courseId
      this.video.chapterId= this.chapterId
      video.saveVideoInfo(this.video).then(response=>{
        this.$message({
        type: 'success',
        message: '保存成功!'
      })
      this.helpSaveVideo()
      })
    },

    editVideo(videoId) {
        this.dialogVideoFormVisible = true
            video.getVideoInfoById(videoId).then(response => {
            this.video = response.data.item
            if(response.data.item.videoOriginalName != '' && response.data.item.videoOriginalName != null ){
              this.fileList = [{'name': this.video.videoOriginalName}]

            }
            })
    },
    //修改小节
    updateDataVideo() {
      video.updateVideoInfoById(this.video).then(response => {
        this.$message({
          type: 'success',
          message: '修改成功!'
        })
        this.helpSaveVideo()
      })
    },
    //删除小节
    removeVideo(videoId) {
      this.$confirm('此操作将永久删除该记录, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        return video.removeById(videoId)
      }).then(() => {
        this.fetchChapterNestedListByCourseId()// 刷新列表
        this.$message({
          type: 'success',
          message: '删除成功!'
        })
      }).catch((response) => { // 失败
        if (response === 'cancel') {
          this.$message({
            type: 'info',
            message: '已取消删除'
          })
        }
      })
    },
    //清空表单刷新页面关闭添加小节弹窗
    helpSaveVideo(){
      this.dialogVideoFormVisible = false// 如果保存成功则关闭对话框
      this.video.title = ''
      this.video.sort = 0
      this.video.free = 0
      this.video.videoSourceId = ''// 重置视频资源id
      this.saveVideoBtnDisabled = false
      this.fetchChapterNestedListByCourseId()// 刷新列表
    },

    
    saveOrUpdate() {
      this.saveBtnDisabled = true
        if (!this.chapter.id) {
          this.saveData()
        } else {
          this.updateData()
        }
    },    

    saveData(){
        this.chapter.courseId = this.courseId
        chapter.save(this.chapter).then(response =>{
              this.$message({
                type: 'success',
                message: '保存成功!'
              })
            //清空表单以及刷新        
            this.helpSave()
        }).catch((response) => {
          this.$message({
            type: 'error',
            message: response.message
          })
      })
    },
    removeChapter(chapterId){
      this.$confirm('此操作将永久删除该记录, 是否继续?', '提示', {
    confirmButtonText: '确定',
    cancelButtonText: '取消',
    type: 'warning'
  }).then(() => {
    return chapter.removeById(chapterId)
  }).then(() => {
    this.fetchChapterNestedListByCourseId()// 刷新列表
    this.$message({
      type: 'success',
      message: '删除成功!'
    })
  }).catch((response) => { // 失败
    if (response === 'cancel') {
      this.$message({
        type: 'info',
        message: '已取消删除'
      })
    } else {
      this.$message({
        type: 'error',
        message: response.message
      })
    }
  })
    },



// 修改章节 
    editChapter(chapterId) {
    this.dialogChapterFormVisible = true
    chapter.getById(chapterId).then(response => {
        this.chapter = response.data.item
    })
},
    updateData() {
      this.chapter.courseId = this.courseId
        chapter.updateById(this.chapter).then(response =>{
              this.$message({
                type: 'success',
                message: '修改成功!'
              })
            //清空表单以及刷新        
            this.helpSave()
        }).catch((response) => {
          this.$message({
            type: 'error',
            message: response.message
          })
      })
    },

    // 清空表单刷新页面关闭添加章节弹窗
    helpSave(){
      this.dialogChapterFormVisible = false// 如果保存成功则关闭对话框
      this.chapter.title = ''// 重置章节标题
      this.chapter.sort = ''
      this.fetchChapterNestedListByCourseId()// 刷新列表
      this.saveBtnDisabled = false
    },


    //阿里云视频点播成功回调
    handleVodUploadSuccess(response, file, fileList) {
      this.video.videoSourceId = response.data.videoId
    },
    //上传视频视图上传多于一个视频
    handleUploadExceed(files, fileList) {
      this.$message.warning('想要重新上传视频，请先删除已上传的视频')
    },

    handleVodUploadSuccess(response, file, fileList) {
        this.video.videoSourceId = response.data.videoId
        this.video.videoOriginalName = file.name;
    },

    //阿里云视频点播删除视频
    beforeVodRemove(file, fileList) {
      return this.$confirm(`确定移除 ${file.name}？`)
    },

   handleVodRemove(file, fileList) {
    console.log(file)
    vod.removeById(this.video.videoSourceId).then(response => {
        this.video.videoSourceId = ''
        this.video.videoOriginalName = ''
        this.fileList = []
        this.$message({
            type: 'success',
            message: response.message
        })
    })
},
    previous() {
      console.log('previous')
      this.$router.push({ path: '/edu/course/info/' + this.courseId  })

    },
    next() {
      console.log('next')
      this.$router.push({ path: '/edu/course/publish/' +this.courseId })
    }
  }
}
</script>   