<template>
  <div class="full" v-if="this.$store.state.Passage.currentPassage.name !== ''">
    <div class="editor-header">
      <h2 class="title">正在编辑： {{this.$store.state.Passage.currentPassage.name}} </h2>
      <el-button type="text" @click="close" class="close">关闭编辑器</el-button>
    </div>
    <el-row class="content">
      <el-col class="full" :span="12">
        <div class="editor-wrap">
          <codemirror class="editor" v-model="code" :options="cmOption" @input="onCmCodeChange">{{code}}</codemirror>
        </div>
      </el-col>
      <el-col class="full" :span="12">
        <div class="preview-wrap">
          <div class="markdown-render" v-html="rendered"></div>
        </div>
      </el-col>
    </el-row>
  </div>
  <pending v-else content="没有正在编辑的专栏！"></pending>
</template>

<script>
import { codemirror } from 'vue-codemirror'
import { clipboard } from 'electron'
import * as renderer from '../utils/RendererFactory'
import * as biliNetwork from '../utils/biliNetwork'
import pending from './subcomponents/Pending'

import 'codemirror/mode/markdown/markdown'
import 'codemirror/addon/selection/active-line'
import 'codemirror/addon/scroll/simplescrollbars.js'

// Theme
import 'codemirror/theme/mdn-like'

export default {
  components: {
    codemirror,
    pending
  },
  data () {
    return {
      code: '',
      cookies: undefined,
      cmOption: {
        mode: 'markdown',
        theme: 'men-like',
        lineNumbers: true,
        line: true,
        styleActiveLine: true,
        lineWrapping: true,
        scrollbarStyle: 'overlay',
        viewportMargin: Infinity,
        extraKeys: {
          'Ctrl-V': async (cm) => {
            let paste, doc = cm.getDoc(), cursor = doc.getCursor()
            const isImage = clipboard.readImage().getBitmap().byteLength !== 0

            if (isImage) {
              paste = 'data:image/png;base64,' + clipboard.readImage().toPNG().toString('base64')
              const result = await biliNetwork.upcover(paste, this.cookies)
              if (result['code'] !== 0) {
                paste = ''
              } else {
                paste = `![](${result['data']['url']})\n`
              }
            } else {
              paste = clipboard.readText()
            }
            doc.replaceRange(paste, cursor)
            this.$store.commit('SET_CONTENT', this.code)
            this.$store.commit('SAVE_PASSAGE', this.code)
          }
        }
      }
    }
  },
  methods: {
    onCmCodeChange () {
      this.$store.commit('SET_CONTENT', this.code)
      this.$store.commit('SAVE_PASSAGE', this.code)
    },
    close () {
      this.$store.commit('SAVE_PASSAGE', this.code)
      this.$store.commit('RESET_PASSAGE')
      // 跳转回专栏管理界面
      this.$router.push('/passages')
      this.$store.commit('SET_PAGE', 1)
    }
  },
  computed: {
    rendered: function () {
      return renderer.getRenderer({module: this.$store.state.Config.config.renderer})(this.code)
    }
  },
  created () {
    this.code = this.$store.state.Passage.currentPassage.text
    this.cookies = this.$store.state.Config.config.cookie
  }
}
</script>

<style>
@import "~codemirror/addon/scroll/simplescrollbars.css";
@import "~codemirror/lib/codemirror.css";

.full {
  height: 100%;
}

.editor-header {
  height: 3%;
  width: 100%;
  display: flex;
  justify-content: space-between;
  font-family: "Helvetica Neue", Helvetica, "PingFang SC", "Hiragino Sans GB",
    "Microsoft YaHei", "微软雅黑", Arial, sans-serif;
}

.title {
  font-size: 18px;
}

.close {
  font-size: 14px;
  display: flex;
  align-content: center;
  justify-content: center;
}

.content {
  height: 97%;
}

.editor-wrap {
  margin-top: 1%;
  margin-bottom: 1%;
  height: 100%;
  width: 100%;
}

.preview-wrap {
  margin-top: 1%;
  margin-bottom: 1%;
  height: 100%;

  border: 1px solid #ccc;
  margin-left: 5px;

  max-height: 100%;
  overflow-x: hidden;
  word-break: break-all;
}

.editor {
  border: 1px solid #ccc;
  height: 100%;
}

.CodeMirror {
  height: 100%;
}

.CodeMirror-scroll {
  overflow-y: hidden;
  overflow-x: auto;
}

.markdown-render {
  display: inline-block;
  font-family: "Helvetica Neue", Helvetica, "PingFang SC", "Hiragino Sans GB",
    "Microsoft YaHei", "微软雅黑", Arial, sans-serif;
}
</style>