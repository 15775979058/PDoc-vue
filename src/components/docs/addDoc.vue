<template>
	<div id="test" class="add-doc">
    <mu-card-title :title="title"/>
    <mu-sub-header><b>标题</b></mu-sub-header>
    <mu-text-field hintText="标题" class="contentInput" v-model="newLabel"/>
    <mu-sub-header><b>描述</b></mu-sub-header>
    <mu-text-field hintText="描述" class="contentInput" v-model="newDesc"/>
    <mu-sub-header><b>markdown文本<mu-switch class="demo-switch" v-model="isMd"/></b> </mu-sub-header>
    <quill-editor v-if="!isMd" v-model="newEditorContent"
      ref="myQuillEditor"
      :options="editorOption"
      class="editorInput">
    </quill-editor> 
<!--     <quill-editor v-if="!isMd" v-model="newEditorContent"
      ref="myQuillEditor"
      :options="editorOption"
      @blur="onEditorBlur($event)"
      @focus="onEditorFocus($event)"
      @ready="onEditorReady($event)"
      class="editorInput">
    </quill-editor>  -->
    <!-- TODOmd -->
    <div id="mdEditor">
      <markdown-editor v-if="isMd" v-model="mdContent" :configs="configs" ref="markdownEditor" class="mdInput" :custom-theme="true"></markdown-editor>
    </div>
    <div class="buttonInput">
      <mu-raised-button label="提交" class="demo-raised-button" @click="addNewDoc" primary/>
      <mu-raised-button label="重置" class="demo-raised-button"/>
    </div>
  </div>	
</template>

<script>
import docApi from '../../api/docApi'
import Store from '../../utils/store'
import { markdownEditor } from 'vue-simplemde'
import editorEvent from '../../assets/common'
export default {
  name: 'add-doc',
  components: {
    markdownEditor
  },
  data () {
    return {
      msg: '这是首页',
      isMd: false,
      newLabel: '',
      newDesc: '',
      newEditorContent: '',
      editorOption: {},
      mdContent: '', // 更新后的md文档内容
      configs: {
        status: false
      },
      currentDocContent: ''
    }
  },
  beforeCreate () {
    if (this.$store.state.user.login == false) {
      this.$router.push('/login')
    }
  },
  mounted () {
    this.editorEvent()
  },
  computed: {
    localUserStorage () {
      // window.localStorege.user转换为json
      return Store.fetch('user')
    },
    editor() {
      return this.$refs.myQuillEditor.quill
    },
    simplemde () {
      return this.$refs.markdownEditor.simplemde
    },
    title () {
      if (this.$route.params.id == 0) {
        return '新增一级文档'
      } else {
        return '新增子级文档'
      }
    }
  },
  methods: {
    addNewDoc () {
      let _this = this;
      let pid = this.$route.params.id;
      let isRoot = 0;
      let mdHtml = null;
      if (pid != 0) {
        isRoot = 1
      }
      if (this.isMd == true) {
        this.currentDocContent = this.mdContent;
        mdHtml = this.simplemde.markdown(this.mdContent);
      } else {
        this.currentDocContent = this.newEditorContent;
      }
      docApi.getSingleDoc(pid)
        .then(function (ret) {
          if (ret.data.code == -1 || ret.data.result.level == 1 || ret.data.result.level == 2) {
            docApi.addDoc(isRoot, pid, _this.newLabel, _this.newDesc, _this.currentDocContent, mdHtml, _this.isMd, _this.$store.state.user.userInfo.email)
              .then(function (res) {
                if (res.data.code == 1) {
                  _this.$notify.success({
                    title: '成功',
                    message: '新增文档成功'
                  })
                  _this.newLabel = '';
                  _this.newDesc = '';
                  _this.newEditorContent = '';
                  _this.mdContent = '';
                  docApi.getDocs(_this.$store.state.user.userInfo.email)
                    .then(function (res) {
                      var docInfo = {
                        newTreeNodes: res.data.result
                      }
                      _this.$store.dispatch('updatedoc', docInfo);
                      _this.$router.push('/docsview/add-doc/' + 0);
                    })
                }
                if (res.data.code == -1) {
                  _this.$notify.error({
                    title: '失败',
                    message: res.data.msg
                  })
                }
              })
          } else if (ret.data.result.level == 3) {
            _this.$notify.error({
              title: '失败',
              message: '文档不能超过三级'
            })
          }
        })
    },
    rootLabelIsExist (label) {
      var rootNodes = this.$store.state.docs.treeNodes;
      for (var i = 0; i < rootNodes.length; i++) {
        if (rootNodes[i].label == label) {
          return false;
        }
      }
    },
    editorEvent () {
      let _this = this
      var mdEditor = document.getElementById('mdEditor');
      editorEvent.editorEvent(mdEditor, _this);
    }
  }
}
</script>

<style lang="stylus" scoped>
.demo-switch
  top 3px
  margin-left 10px
.buttonInput
  top 0px
  margin-left 15px
.searchInput
  margin auto
  width 100%
.contentInput
  width 100%
  margin-left 15px
.editorInput
  width 100%
  height 400px
  margin-left 15px
  margin-bottom 70px
.mdInput
  width 100%
  margin-left 15px
  margin-bottom 5px
</style>
