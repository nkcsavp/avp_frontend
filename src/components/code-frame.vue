<template>
  <div v-loading="load" class="code-frame">
    <el-alert
        v-show="wrong"
        :description="description"
        show-icon
        title="Error"
        type="error"
    ></el-alert>
    <el-form ref="codeForm" :hide-required-asterisk="true" :model="codeForm" :rules="rules" label-position="left" label-width="100px"
             status-icon>
      <el-tooltip content="选择运行模式" placement="left" trigger="hover">
        <el-form-item label="Mode" prop="mode">
          <el-select v-model="codeForm.mode" placeholder="Mode">
            <el-option label="Array" value="array"></el-option>
            <el-option label="Tree" value="tree"></el-option>
            <el-option label="Graph" value="graph"></el-option>
          </el-select>
        </el-form-item>
      </el-tooltip>
      <el-tooltip content="选择语言" placement="left" trigger="hover">
      <el-form-item label="Language" prop="lang">
          <el-select v-model="codeForm.lang" placeholder="Language:">
            <el-option label="Java" value="java"></el-option>
            <el-option label="Python" value="python"></el-option>
            <el-option label="C++" value="cpp"></el-option>
          </el-select>
        </el-form-item>
      </el-tooltip>
      <el-tooltip content="设置本次执行的标签信息" placement="left" trigger="hover">
        <el-form-item label="Tag" prop="tag">
          <el-input v-model="codeForm.tag"></el-input>
        </el-form-item>
      </el-tooltip>
      <el-tooltip content="输入测试样例(改变Mode会清空)" placement="left" trigger="click">
        <el-form-item label="Sample" prop="sample">
          <el-input v-model="codeForm.sample"></el-input>
        </el-form-item>
      </el-tooltip>
      <el-tooltip  content="输入图的边信息(a:b,c:d)" placement="left" trigger="click">
        <el-form-item v-show="codeForm.mode === 'graph'" label="Relation" prop="relation">
          <el-input v-model="codeForm.relation"></el-input>
        </el-form-item>
      </el-tooltip>
      <el-tooltip content="输入代码(改变Mode或Language会清空)" placement="top" trigger="click">
        <el-form-item label="Code" prop="codes">
          <div class="editor" style="width: 100%">
            <v-ace-editor
                v-model:value="codeForm.codes"
                :lang="codeForm.lang"
                style="height: 300px"
                theme="xcode"
            />
          </div>
        </el-form-item>
      </el-tooltip>
      <el-form-item>
        <el-button type="primary" @click="submit">Submit</el-button>
      </el-form-item>
    </el-form>
  </div>
</template>

<script>
import axios from "axios"
import {VAceEditor} from 'vue3-ace-editor'
import 'ace-builds/src-noconflict/mode-java'
import 'ace-builds/src-noconflict/mode-c_cpp'
import 'ace-builds/src-noconflict/mode-python'
import 'ace-builds/src-noconflict/theme-xcode'
import {useStore} from "vuex"
import {ref, watch} from "vue";
import router from "../router";
import {ElNotification} from "element-plus";

export default {
  name: "code-frame",
  emits: [
    'submitted'
  ],
  props: {
    'initCode': String,
    'initMode': String,
    'initLang': String,
    'initSample': String,
    'initTag': String,
    'initRelation': String
  },
  data() {
    const config = {
      'java': {
        'tree': "BinaryTree bt = new BinaryTree();\n" +
            "BinaryTreeNode root = bt.getRoot();\n",
        'array': "DataList data = new DataList();\n",
        'graph': "Graph g = new Graph();"
      },
      'cpp': {
        'array': "int main(){\n" +
            "\tDataList data;\n" +
            "}",
        'tree': "int main(){\n" +
            "\tBinaryTree binaryTree;\n" +
            "\tBinaryTreeNode* root = binaryTree.getRoot();\n" +
            "}",
        'graph': "int main(){\n" +
            "\tGraph g;\n" +
            "}",
      },
      'python': {
        'array': "data = DataList()\n",
        'tree': "bt = BinaryTree()\n" +
            "root = bt.root\n",
        'graph': "g = Graph()"
      },
      'array': {
        sample: '7,6,2,4,3,1,5',
        reg: /^([0-9]+,)*([0-9]+)$/
      },
      'tree': {
        sample: '1,2,3,4,0,6',
        reg: /^([0-9]+,)*([0-9]+)$/
      },
      'graph': {
        sample: '1,2,3,4',
        reg: /^([0-9]+,)*([0-9]+)$/,
      }
    }
    const aceMap = {
      'java': 'java',
      'python': 'python',
      'cpp': 'c_cpp'
    }

    const store = useStore()
    let mode = ref(this.initMode?this.initMode:'array')
    let lang = ref(this.initLang?this.initLang:'java')
    let load = ref(false)
    let sample = ref(this.initSample?this.initSample:config[mode.value].sample)
    let tag = ref(this.initTag?this.initTag:("trial"))
    let relation = ref(this.initRelation?this.initRelation:"0:1,1:2")
    let codes = ref(this.initCode?this.initCode:config[lang.value][mode.value])
    let wrong = ref(false);
    let description = ref(null);

    const submit = () => {
      this.$refs.codeForm.validate((valid) => {
        if (!valid) {
          return;
        }
        load.value = true
        let url
        let relationMatrix
        if(this.codeForm.mode === "graph"){
          const len = ( this.codeForm.sample.length + 1 ) / 2
          relationMatrix = new Array(len * len).fill(0)
          let splitRelation = this.codeForm.relation.split(",")
          splitRelation.forEach((e)=>{
            let edge = e.split(":")
            relationMatrix[Number(edge[0]) * len + Number(edge[1])] = 1
          })
          url = "submit?lang=" + this.codeForm.lang + "&mode=" + this.codeForm.mode + "&sample=" + this.codeForm.sample.replaceAll('x', '0') + "&tag=" + this.codeForm.tag + "&relation=" + relationMatrix.join(',')
        }else{
          url = "submit?lang=" + this.codeForm.lang + "&mode=" + this.codeForm.mode + "&sample=" + this.codeForm.sample.replaceAll('x', '0') + "&tag=" + this.codeForm.tag
        }
        axios({
          url: url,
          method: 'POST',
          headers: {'content-type': 'text/plain'},
          data: this.codeForm.codes
        }).then((res) => {
          load.value = false
          const ret = [sample.value.split(',').map((val)=>val === '0'?undefined:Number(val)), res.data['msg'].split(':'), mode.value, codes.value, lang.value, sample.value, tag.value, relation.value, relationMatrix]
          this.$emit('submitted', ret)
        }).catch((err) => {
          load.value = false
          if (err.response.status === 403) {
            ElNotification({
              title: 'Error',
              message: err.response.data['msg'],
              type: 'error',
            })
            store.dispatch("LogOut")
            router.push("/login")
          } else if (err.response.status === 400) {
            wrong.value = true
            description.value = err.response.data['msg']
          } else {
            wrong.value = true
            description.value = "未知错误，可能是服务器出现故障"
          }
        })
      })
    }
    const checkSample = (rule, value, callback) => {
      let samplePattern = config[mode.value].reg;
      if (value === "") {
        return callback(new Error('[注意]请输入测试样例'))
      }
      setTimeout(() => {
        if (!samplePattern.test(value)) {
          callback(new Error('[注意]测试样例格式： \"' + config[mode.value].sample + '\"'))
        } else {
          callback()
        }
      }, 100)
    }
    const checkTag = (rule, value, callback) => {
      setTimeout(() => {
        if (value.length === 0 || value.length > 200) {
          callback(new Error('[注意]标签长度为1-200个字符'))
        } else {
          callback()
        }
      }, 100)
    }
    const checkRelation = (rule, value, callback) => {
      if(mode.value !== 'graph') callback()
      const relationPattern = /^([0-9]+:[0-9]+,)*$/;
      setTimeout(() => {
        if (!relationPattern.test(value + ',') && value.length !== 0) {
          callback(new Error('[注意]测试样例格式: \"0:1,1:2\"'))
        } else {
          callback()
        }
      }, 100)
    }

    watch(mode, (newMode, old) => {
      sample.value = config[mode.value].sample
      codes.value = config[lang.value][mode.value]
    })
    watch(lang, (newLang, old) => {
      codes.value = config[lang.value][mode.value]
    })
    return {
      submit,
      load,
      wrong,
      description,
      aceMap,
      codeForm: {
        mode,
        lang,
        codes,
        sample,
        tag,
        relation
      },
      rules: {
        sample: [{required: true, validator: checkSample, trigger: 'blur'}],
        lang: [{required: true, trigger: 'blur'}],
        codes: [{required: true, trigger: 'blur'}],
        mode: [{required: true, trigger: 'blur'}],
        tag: [{required: true, validator: checkTag, trigger: 'blur'}],
        relation: [{required: true, validator: checkRelation, trigger: 'blur'}]
      }
    }
  },
  components: {
    VAceEditor
  }
}
</script>

<style scoped>
.code-frame {
  padding: 30px;
  border: 1px solid var(--el-border-color-base);
  box-shadow: var(--el-box-shadow-light);
  border-radius: var(--el-border-radius-base);
  margin-top: 30px;
}

.editor {
  border-radius: var(--el-border-radius-base);
  padding: 2px;
  border: 1px solid var(--el-border-color-base);
  box-shadow: var(--el-box-shadow-light);
}

.el-alert {
  margin-bottom: 15px;
}
</style>