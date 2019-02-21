<template>
  <div class="blog-container">
    <div class="blog-pages">
      <div class="col-md-12 panel">
        <div class="panel-body">
          <h2 class="text-center">{{ articleId ? '编辑文章' : '创作文章' }}</h2>
          <hr>
          <div data-validator-form>
            <div class="form-group">
              <input v-model.trim="title" v-validator:blur.required="{ title: '标题' }" type="text" class="form-control"
                     placeholder="请填写标题">
            </div>
            <div class="form-group">
              <textarea id="editor"></textarea>
            </div>
            <br>
            <div class="form-group">
              <button class="btn btn-primary" @click="post" type="submit">发 布</button>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
  import SimpleMDE from 'simplemde'
  import hljs from 'highlight.js'
  import ls from '@/utils/localStorage'

  window.hljs = hljs

  export default {
    name: 'Create',
    // 添加相关数据
    data() {
      return {
        title: '', // 文章标题
        content: '', // 文章内容
        articleId: undefined // 文章 ID
      }
    },
    beforeRouteEnter(to, from, next) {
      next(vm => {
        // 确认渲染组件的对应路由时，设置 articleId
        vm.setArticleId(vm.$route.params.articleId)
      })
    },
    // 在离开该组件的对应路由前
    beforeRouteLeave(to, from, next) {
      // 清空自动保存的文章数据
      this.clearData()
      next()
    },
    watch: {
      // 监听路由参数的变化
      '$route'(to) {
        // 清空自动保存的文章数据
        this.clearData()
        //设置 articleId
        this.setArticleId(to.params.articleId)
      }
    },
    mounted() {
      const simplemde = new SimpleMDE({
        element: document.querySelector('#editor'),
        placeholder: '请使用 Markdown 格式书写 ;-)，代码片段黏贴时请注意使用高亮语法。',
        spellChecker: false,
        autoDownloadFontAwesome: false,
        autosave: {
          enabled: true,
          uniqueId: 'vuejs-essential'
        },
        renderingConfig: {
          codeSyntaxHighlighting: true
        }
      })

      // 监听编辑器的 change 事件
      simplemde.codemirror.on('change', () => {
        // 将改变后的值赋给文章内容
        this.content = simplemde.value()
      })

      // 将 simplemde 添加到当前实例，以在其他地方调用
      this.simplemde = simplemde

    },
    methods: {
      // 编辑器只会自动保存文章的内容，我们需要自己保存文章的标题
      saveTitle() {
        ls.setItem('smde_title', this.title)
      },
      // 初始化标题和内容
      fillContent(articleId) {
        const simplemde = this.simplemde
        const smde_title = ls.getItem('smde_title')

        if (articleId !== undefined) {
          const article = this.$store.getters.getArticleById(articleId)

          if (article) {
            // 获取文章的标题和内容
            const {title, content} = article

            this.title = smde_title || title
            this.content = simplemde.value() || content
            simplemde.value(this.content)
          }
        } else {
          this.title = smde_title
          this.content = simplemde.value()
        }
      },
      // 发布
      post() {
        const title = this.title
        const content = this.content

        // 如果标题和内容都不为空，提交发布
        if (title !== '' && content.trim() !== '') {
          const article = {
            title,
            content
          }

          // 为 => 分发 post 事件，并附带参数 { article }
          this.$store.dispatch('post', {article})

          // 清除数据
          this.clearData()
        }
      },
      // 清除数据
      clearData() {
        this.title = ''
        ls.removeItem('smde_title')
        // 清除编辑的内容
        this.simplemde.value('')
        // 清除编辑器自动保存的内容
        this.simplemde.clearAutosavedValue()
      },
      // 设置 articleId
      setArticleId(articleId) {
        const localArticleId = ls.getItem('articleId')

        if (articleId !== undefined && !(articleId === localArticleId)) {
          this.clearData()
        }
        // 设置当前实例的 articleId
        this.articleId = articleId
        // 填充文章数据
        this.fillContent(articleId)
        // 在 localStorage 保存一个 articleId
        ls.setItem('articleId', articleId)
      }
    }
  }
</script>

<style lang="scss">
  .blog-container {
    max-width: 980px;
    margin: 0 auto;
    margin-top: 20px;
  }

  textarea {
    height: 200px;
  }

  @import 'simplemde/dist/simplemde.min.css';
  @import 'highlight.js/styles/paraiso-dark.css';
</style>
