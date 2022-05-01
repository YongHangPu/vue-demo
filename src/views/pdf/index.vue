<template>
  <div style="padding: 30px">
    <div>
      文件列表
      <ul class="imglist">
        <li v-for="item in fileList" :key="item.id" class="imgitem">
          <span class="ltext">{{ item.fileName }}</span>
          <span class="rbut">
            <el-button type="text" @click="preview(item)">预览</el-button>
          </span>
        </li>
      </ul>
    </div>
    <el-dialog title="文件预览" :visible.sync="dialogVisible" width="50%" :before-close="handleClose">
      <div v-if="fileType === 'pdf'">
        <!-- 加载全部页面的PDF是一个for循环,不能指定用来打印的ref -->
        <!-- <Pdf v-for="i in numPages" :key="i" :src="src" :page="i" /> -->
        <div class="toolbar">
          <el-button size="mini" round @click.stop="prePage">上一页</el-button>
          <el-button size="mini" round @click.stop="nextPage">下一页</el-button>
          <div class="progress">{{ pageNum }}/{{ pageTotalNum }}</div>
          <el-button size="mini" round @touchstart="idx = 0" @touchend="idx = -1" @click="scaleD">放大</el-button>
          <el-button size="mini" round @touchstart="idx = 1" @touchend="idx = -1" @click="scaleX">缩小</el-button>
          <!-- <el-button size="mini" round @click.stop="clock">顺时针</el-button>
          <el-button size="mini" round @click.stop="counterClock">逆时针</el-button> -->
          <el-button size="mini" round @click="fileDownload(pdfUrl, currentFile.fileName)">下载</el-button>
        </div>
        <pdf
          ref="pdf"
          :src="pdfUrl"
          :page="pageNum"
          :rotate="pageRotate"
          @num-pages="pageTotalNum = $event"
          @link-clicked="page = $event"
        />
      </div>
      <!-- 音频 -->
      <div v-if="dialogVisible && fileType === 'audio'">
        <audio-player
          class="audio-custom"
          ref="audioPlayer"
          :audio-list="audioURL"
          :show-prev-button="false"
          :show-next-button="false"
          theme-color="#000"
        />
      </div>
      <!-- 视频 -->
      <div v-if="dialogVisible && fileType === 'video'">
        <video-player :options="playerOptions" />
      </div>
    </el-dialog>
  </div>
</template>

<script>
// pdf
import Pdf from 'vue-pdf'
// 音频
import audioPlayer from '@liripeng/vue-audio-player'
// 视频
import videoPlayer from './videoPlayer.vue'

let origin = window.location.origin
export default {
  components: {
    Pdf,
    audioPlayer,
    videoPlayer
  },
  data() {
    return {
      fileList: [
        {
          id: 1,
          fileName: '应收账款',
          fileUrl: `${origin}/test1.pdf`,
        },
        {
          id: 2,
          fileName: '测试PDF',
          fileUrl: `${origin}/test2.pdf`,
        },
        {
          id: 3,
          fileName: '音频aac',
          fileUrl: `${origin}/音频.aac`,
        },
        {
          id: 4,
          fileName: '音频ogg',
          fileUrl: `${origin}/音频.ogg`,
        },
        {
          id: 5,
          fileName: '音频mp3',
          fileUrl: `${origin}/音频.mp3`,
        },
        {
          id: 11,
          fileName: '音频wav',
          fileUrl: `${origin}/音频.wav`,
        },
        {
          id: 6,
          fileName: '视频mp4',
          fileUrl: `${origin}/mp4.mp4`,
        },
        {
          id: 7,
          fileName: '视频mkv',
          fileUrl: `${origin}/mkv.mkv`,
        },
        {
          id: 8,
          fileName: '视频mov',
          fileUrl: `${origin}/mov.mov`,
        },
        {
          id: 9,
          fileName: '视频ogv',
          fileUrl: `${origin}/ogv.ogv`,
        },
        {
          id: 10,
          fileName: '视频webm',
          fileUrl: `${origin}/webm.webm`,
        },
      ],
      numPages: undefined,
      dialogVisible: false,
      src: '',
      // 音频文件类型
      audioList: ['m4a', 'mp3', 'aac', 'caf', 'flac', 'oga', 'wav', 'ogg'],
      videoList: ['mp4', 'ogv', 'mov', 'm4v', 'mkv', 'webm', 'm3u8'],
      imgList: [],
      // video.js支持的视频格式
      MimetypesKind: {
        ogv: 'video/ogg',
        webm: 'video/webm',
        mp4: 'video/mp4',
        mov: 'video/mp4',
        m4v: 'video/mp4',
        mkv: 'video/mp4',
        m3u8: 'application/x-mpegURL',
      },
      fileType: '',
      currentFile: '',
      options: {
        i18n: {
          speed: '速度',
          normal: '正常',
        },
      },
      // 视频播放器配置
      playerOptions: {
        sources: [],
      },
      audioURL: [],
      pdfUrl: '',
      pageNum: 1,
      pageTotalNum: 1,
      pageRotate: 0,
      scale: 100, //放大系数
      idx: -1,
    }
  },
  created() {},
  methods: {
    // 预览
    preview(item) {
      this.dialogVisible = true
      this.currentFile = item
      this.fileType = this.getFileType(item.fileUrl)
      // 暂时用switch，后续根据情况再替换
      switch (this.fileType) {
        case 'pdf':
          // this.src = Pdf.createLoadingTask(item.fileUrl)
          // this.src.promise.then(pdf => {
          //   this.numPages = pdf.numPages
          // })
          this.pdfUrl = item.fileUrl
          break
        case 'video':
          let file = {
            src: item.fileUrl,
            type: this.getMimetype(item.fileUrl),
          }
          this.playerOptions.sources.push(file)
          break
        case 'audio':
          this.audioURL = [item.fileUrl]
          break
      }
      console.log(this.src, this.audioURL)
    },
    // 关闭弹框
    handleClose() {
      this.playerOptions.sources.length = 0
      this.src = ''
      this.pageNum = 1
      this.pageTotalNum = 1
      // 暂停播放
      this.$refs.audioPlayer && this.$refs.audioPlayer.pause()
      this.dialogVisible = false
    },
    // 根据文件fileUrl获取文件类型，判断是pdf还是音频或者视频
    getFileType(fileUrl) {
      let fileType = fileUrl.replace(/.+\./, '')
      if (this.audioList.includes(fileType)) {
        return 'audio'
      } else if (this.videoList.includes(fileType)) {
        return 'video'
      } else {
        return 'pdf'
      }
    },
    // 返回传递的文件名的扩展名
    getFileExtension(path) {
      if (typeof path === 'string') {
        let splitPathRe = /^(\/?)([\s\S]*?)((?:\.{1,2}|[^\/]+?)(\.([^\.\/\?]+)))(?:[\/]*|[\?].*)$/
        let pathParts = splitPathRe.exec(path)
        if (pathParts) {
          return pathParts.pop().toLowerCase()
        }
      }
      return ''
    },
    // 获取音频或视频type格式
    getMimetype(src = '') {
      var ext = this.getFileExtension(src)
      var mimetype = this.MimetypesKind[ext.toLowerCase()]
      return mimetype || ''
    },

    // pdf方法
    //下载PDF
    fileDownload(src, fileName) {
      var x = new XMLHttpRequest()
      x.open('GET', src, true)
      x.responseType = 'blob'
      x.onload = function (e) {
        var blob = x.response
        var a = document.createElement('a')
        a.download = fileName
        a.href = URL.createObjectURL(blob)
        document.body.append(a)
        a.click()
        document.body.removeChild(a)
      }
      x.send()
    },
    //放大
    scaleD() {
      this.scale += 5
      this.$refs.pdf.$el.style.width = parseInt(this.scale) + '%'
    },

    //缩小
    scaleX() {
      // if (this.scale == 100) {
      //   return
      // }
      this.scale += -5
      this.$refs.pdf.$el.style.width = parseInt(this.scale) + '%'
    },
    prePage() {
      var p = this.pageNum
      p = p > 1 ? p - 1 : this.pageTotalNum
      this.pageNum = p
    },
    nextPage() {
      var p = this.pageNum
      p = p < this.pageTotalNum ? p + 1 : 1
      this.pageNum = p
    },
    clock() {
      this.pageRotate += 90
    },
    counterClock() {
      this.pageRotate -= 90
    },
  },
}
</script>
<style lang="less" scoped>
.imglist {
  width: 500px;
  padding-top: 20px;
  .imgitem {
    height: 35px;
    overflow: hidden;
  }
  .ltext {
    line-height: 35px;
  }
  .rbut {
    float: right;
  }
}

/* 音频自定义样式 */
.audio-custom {
  ::v-deep {
    .audio__play-rate__dropdown {
      width: 30px;
      text-align: center;
    }
  }
}

/* pdf头部操作项样式 */
.toolbar {
  display: flex;
  justify-content: center;
  align-items: center;

  .progress {
    margin: 0 20px;
    color: blue;
    font-size: 20px;
  }
}
</style>
