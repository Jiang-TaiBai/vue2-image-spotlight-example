<script>
import DataOsiModel from "@/assets/js/Data-OsiModel";
import DataSpongeBob from "@/assets/js/Data-SpongeBob";
import DataOrgans from "@/assets/js/Data-Organs";
import ConfigurationEditorCustom from "@/assets/js/Configuration-Editor-Custom";
import ConfigurationRendererCustom from "@/assets/js/Configuration-Renderer-Custom";
import DataIPhone14 from "@/assets/js/Data-iPhone14";

import VersionText from "@/assets/js/VersionText";

export default {
  data() {
    return {
      datas: [
        DataSpongeBob,
        DataOsiModel,
        DataOrgans,
        DataIPhone14,
      ],
      ConfigurationEditorCustom: ConfigurationEditorCustom,
      ConfigurationRendererCustom: ConfigurationRendererCustom,

      usingData: {
        index: 0,
        imgUrl: "",
        allAreaInfos: [],
        allPoints: [],
      },
      exportData: {},

      InformationAboutProject: {
        forVue2Github: "https://github.com/Jiang-TaiBai/vue2-image-spotlight",
        forVue3Github: null,
        forVue2Gitee: "https://gitee.com/jiang-taibai/vue2-image-spotlight",
        forVue3Gitee: null,
        document: "https://jiang-taibai.github.io/vue2-image-spotlight/",
        developer: "Jiang Liu",
      }
    }
  },
  computed: {},
  methods: {
    // 完成编辑后需要更新数据
    onAccomplish(data) {
      this.exportData = data
    },
    // 添加图片
    beforePictureUpload(file) {
      const isValidFileType =
          file.type === 'image/jpeg' ||
          file.type === 'image/jpg' ||
          file.type === 'image/png';
      const isLt20M = file.size / 1024 / 1024 < 20;
      if (!isValidFileType) {
        this.$message.error('上传的图片只能是 jpeg/jpg/png 格式!');
      }
      if (!isLt20M) {
        this.$message.error('上传的图片大小不能超过 20MB!');
      }
      // console.log(URL.createObjectURL(file))
      this.datas.push({
        imgUrl: URL.createObjectURL(file),
        allAreaInfos: [],
        allPoints: [],
      })

      return isValidFileType && isLt20M;
    },
    // 选择图片
    checkPicture(index) {
      this.usingData = {
        index: index,
        ...this.datas[index]
      }
      this.exportData = {
        ...this.datas[index]
      }
    },
    // 删除图片列表中的某一个
    deletePicture(index) {
      // 如果删除的就是当前正在使用的
      if (index === this.usingData.index) {
        // 如果当前正在使用的是第一个
        if (index === 0) {
          // 如果只有一个，不能再删了
          if (this.datas.length === 1) {
            this.$message({
              message: '不能再删啦~',
              type: 'warning'
            });
          } else {
            // 如果不只有一个
            this.datas.splice(index, 1)
            this.usingData = {
              index: 0,
              ...this.datas[0]
            }
          }
        }
        // 当前使用的不是第一个，那么就需要重新选到第一个
        else {
          this.datas.splice(index, 1)
          this.usingData = {
            index: 0,
            ...this.datas[0]
          }
        }
      }
      // 如果删除的在当前使用的后面，则对此无影响
      if (index > this.usingData.index) {
        this.datas.splice(index, 1)
      }
      // 但是如果删除的是在当前使用的前面，那么就需要将当前使用的坐标-1
      if (index < this.usingData.index) {
        this.datas.splice(index, 1)
        let oldIndex = this.usingData.index
        this.usingData = {
          index: oldIndex - 1,
          ...this.datas[oldIndex] - 1
        }
      }

      this.exportData = {
        imgUrl: this.usingData.imgUrl,
        allAreaInfos: this.usingData.allAreaInfos,
        allPoints: this.usingData.allPoints,
      }
    },
    // 将exportData复制到粘贴板
    copyExportData() {
      navigator.clipboard.writeText(JSON.stringify(this.exportData, null, 2))
      this.$message({
        type: "success",
        message: "复制成功",
      })
    },
  },
  // 数据更新需要在mount之前
  beforeMount() {
    this.usingData = {
      index: 0,
      imgUrl: this.datas[0].imgUrl,
      allAreaInfos: this.datas[0].allAreaInfos,
      allPoints: this.datas[0].allPoints,
    }
    this.exportData = {
      imgUrl: this.usingData.imgUrl,
      allAreaInfos: this.usingData.allAreaInfos,
      allPoints: this.usingData.allPoints,
    }
  },
  mounted() {
    console.log(VersionText)
    let that = this
    this.$refs.scroll.addEventListener('wheel', (event) => {
      event.preventDefault()
      that.$refs.scroll.scrollLeft += event.deltaY
    })
  }
}
</script>

<template>
  <div id="app">
    <div style="height: 30px; width: 100%"/>
    <div ref="scroll" class="scroll">
      <div class="img-item" v-for="(data, index) in datas" :key="index"
           @click="checkPicture(index)"
           :style="{backgroundImage: `url(${require('@/assets/imgs/transparent-background.png')})`}">
        <div class="button-group">
          <el-button-group>
            <el-button :type="(usingData.index===index)?'primary':'info'" :plain="!(usingData.index===index)"
                       icon="el-icon-check"
                       @click="checkPicture(index)"
                       size="mini"></el-button>
            <el-button type="danger" plain icon="el-icon-delete"
                       @click="deletePicture(index)"
                       size="mini"></el-button>
          </el-button-group>
        </div>
        <el-image style="width: 150px; height: 150px" :src="data.imgUrl" fit="contain"></el-image>
      </div>
      <div class="img-item">
        <div>
          <el-upload
              class="uploader"
              action=""
              :show-file-list="false"
              :before-upload="beforePictureUpload"
              accept="image/jpeg,image/jpg,image/png">
            <i class="el-icon-plus uploader-icon" style="line-height: 150px;"></i>
          </el-upload>
        </div>
      </div>
    </div>
    <div id="main-area">
      <div id="editor-area">
        <image-spotlight-editor
            :img-url="usingData.imgUrl"
            :all-points="usingData.allPoints"
            :all-area-infos="usingData.allAreaInfos"

            @accomplish="onAccomplish"

            :canvas-width="ConfigurationEditorCustom.canvasWidth"
            :editing-width="ConfigurationEditorCustom.editingWidth"
            :height="ConfigurationEditorCustom.height"

            :selecting-areaStyle="ConfigurationEditorCustom.selectingAreaStyle"
            :drawing-areaStyle="ConfigurationEditorCustom.drawingAreaStyle"
            :other-areaStyle="ConfigurationEditorCustom.otherAreaStyle"

            :rule="ConfigurationEditorCustom.rule"
            :background-image="ConfigurationEditorCustom.backgroundImage"
            :editing-area-background="ConfigurationEditorCustom.editingAreaBackground"
            :area-list-background="ConfigurationEditorCustom.areaListBackground"
            :area-list-item-background="ConfigurationEditorCustom.areaListItemBackground"
            :area-list-item-background-after-hover="ConfigurationEditorCustom.areaListItemBackgroundAfterHover"
            :button-names="ConfigurationEditorCustom.buttonNames"
        />
        <div style="position: relative">
          <el-button type="info" style="position: absolute; top: 10px; right: 10px; opacity: 20%" size="mini"
                     @click="copyExportData">Copy
          </el-button>
          <div class="export-data">{{ JSON.stringify(exportData, null, 2) }}</div>
        </div>
      </div>
      <div id="renderer-area">
        <image-spotlight-renderer
            :img-url="usingData.imgUrl"
            :all-area-infos="usingData.allAreaInfos"
            :all-points="usingData.allPoints"

            :canvas-width="ConfigurationRendererCustom.canvasWidth"
            :canvas-height="ConfigurationRendererCustom.canvasHeight"
            :mask-background="ConfigurationRendererCustom.maskBackground"
            :animation-time="ConfigurationRendererCustom.animationTime"
            :show-description-box="ConfigurationRendererCustom.showDescriptionBox"
            :description-box-style="ConfigurationRendererCustom.descriptionBoxStyle"
        />
      </div>
    </div>
    <div id="footer">
      <div style="width: 80vw; margin: 0 auto">
        <p style="text-align: center;"><span
            style="font-size: x-large; font-weight: bolder">🎈Image Spotlight(I.S.)</span><span>&nbsp;&nbsp;在线体验版</span>
        </p>
        <div style="text-align: center">
          <el-link class="text-link" :href="InformationAboutProject.document" :underline="false" type="info">
            📜Document
          </el-link>
        </div>
        <div style="text-align: center">
          <el-link class="text-link" :href="InformationAboutProject.forVue2Github" :underline="false" type="info">
            🗂️I.S. For Vue2(GitHub)
          </el-link>
          <el-link class="text-link" :href="InformationAboutProject.forVue2Gitee" :underline="false" type="info">
            🗂️I.S.
            For Vue2(Gitee)
          </el-link>
        </div>
        <div style="text-align: center">
          <el-link class="text-link" :underline="false" @click="$message('待完善……')" type="info">🗂️I.S. For
            Vue3(GitHub)
          </el-link>
          <el-link class="text-link" :underline="false" @click="$message('待完善……')" type="info">🗂️I.S. For
            Vue3(Gitee)
          </el-link>
        </div>
        <div style="text-align: center; margin-top: 30px">
          <el-link class="text-link" :underline="false" type="info">Copyright © 2023 Jiang Liu.
          </el-link>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
#app {
  /*background-image: url("../src/assets/imgs/bg/double-bubble-outline.webp");*/
  /*background-image: url("../src/assets/imgs/bg/canadian-dollar.webp");*/
  background-image: url("../src/assets/imgs/bg/y-so-serious-white.png");
  min-width: 780px;
}

.scroll {
  display: flex;
  overflow: auto;
  width: 80vw;
  background: rgba(0, 0, 0, 0.4);
  padding: 8px;
  margin: 0 auto;
  border-radius: 8px;
}

.scroll .img-item {
  position: relative;
  width: 150px;
  height: 150px;
  margin-right: 10px;
  text-align: center;
  flex-shrink: 0;
  cursor: pointer;
}

.scroll .img-item .button-group {
  position: absolute;
  top: 8px;
  right: 0;
  padding: 0 10px;
  opacity: 80%;
  z-index: 999;
}

/* 主要区域，即编辑器与渲染器 */
#main-area {
  margin-top: 16px;
  display: flex;
  justify-content: center;
  flex-wrap: wrap;
  margin-bottom: 40px;
}

#main-area #editor-area, #main-area #renderer-area {
  border-radius: 8px;
  padding: 20px;
  background: rgba(255, 255, 255, 0.6);
  box-shadow: rgba(149, 157, 165, 0.2) 0px 8px 24px;
  transition: all .2s;
}

#main-area #editor-area:hover, #main-area #renderer-area:hover {
  box-shadow: rgba(17, 12, 46, 0.15) 0px 48px 100px 0px;
}

@media screen and (max-width: 1403px) {
  #main-area #editor-area {
    margin-right: 0px;
    margin-bottom: 16px;
  }
}

#main-area #editor-area {
  margin-right: 16px;
}

#main-area #renderer-area {
}

/* 导出数据栏样式 */
.export-data {
  margin-top: 8px;
  width: 568px;
  max-height: 300px;
  overflow: auto;
  border-radius: 8px;
  padding: 20px;
  background: #371722;
  color: #BBAB9B;
  white-space: pre-wrap;
}

#footer {
  min-height: 100px;
  background: #1F3B34;
  color: #FFBC00;
  padding: 50px 50px 20px;
}

#footer .text-link {
  font-size: large;
  font-weight: bold;
  margin: 10px 20px;
}
</style>
<style>
.uploader {
  background: #F8EBEE;
  width: 150px;
  height: 150px;
  border-radius: 8px;
  transition: background-color .2s;
}

.uploader:hover {
  background: #d7ccce;
}

.uploader .el-upload {
  border-radius: 8px;
  cursor: pointer;
  position: relative;
  overflow: hidden;
}

.uploader-icon {
  font-size: 28px;
  color: #8c939d;
  width: 150px;
  height: 150px;
  text-align: center;
  transition: color .2s;
}

.uploader-icon:hover {
  color: #fdf7fa;
}
</style>