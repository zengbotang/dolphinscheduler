/*
 * Licensed to the Apache Software Foundation (ASF) under one or more
 * contributor license agreements.  See the NOTICE file distributed with
 * this work for additional information regarding copyright ownership.
 * The ASF licenses this file to You under the Apache License, Version 2.0
 * (the "License"); you may not use this file except in compliance with
 * the License.  You may obtain a copy of the License at
 *
 *    http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */
<template>
  <div class="shell-model">
    <m-list-box>
      <div slot="text">{{$t('Script')}}</div>
      <div slot="content">
        <div class="form-mirror">
          <textarea
            id="code-shell-mirror"
            name="code-shell-mirror"
            style="opacity: 0">
          </textarea>
          <a class="ans-modal-box-max">
            <em class="el-icon-full-screen" @click="setEditorVal"></em>
          </a>
        </div>
      </div>
    </m-list-box>
    <m-list-box>
      <div slot="text">{{$t('Resources')}}</div>
      <div slot="content">
        <treeselect  v-model="resourceList" :multiple="true" maxHeight="200" :options="options" :normalizer="normalizer" :disabled="isDetails" :value-consists-of="valueConsistsOf" :placeholder="$t('Please select resources')">
          <div slot="value-label" slot-scope="{ node }">{{ node.raw.fullName }} <span  class="copy-path" @mousedown="_copyPath($event, node)" >&nbsp; <em class="el-icon-copy-document" data-container="body"  data-toggle="tooltip" :title="$t('Copy path')" ></em> &nbsp;  </span></div>
        </treeselect>
      </div>
    </m-list-box>
    <m-list-box>
      <div slot="text">{{$t('Custom Parameters')}}</div>
      <div slot="content">
        <m-local-params
                ref="refLocalParams"
                @on-local-params="_onLocalParams"
                :udp-list="localParams"
                :hide="true">
        </m-local-params>
      </div>
    </m-list-box>
    <el-dialog
      :visible.sync="scriptBoxDialog"
      append-to-body="true"
      width="80%">
      <m-script-box :item="item" @getSriptBoxValue="getSriptBoxValue" @closeAble="closeAble"></m-script-box>
    </el-dialog>
  </div>
</template>
<script>
  import _ from 'lodash'
  import i18n from '@/module/i18n'
  import mListBox from './_source/listBox'
  import mScriptBox from './_source/scriptBox'
  import mLocalParams from './_source/localParams'
  import disabledState from '@/module/mixin/disabledState'
  import Treeselect from '@riophae/vue-treeselect'
  import '@riophae/vue-treeselect/dist/vue-treeselect.css'
  import codemirror from '@/conf/home/pages/resource/pages/file/pages/_source/codemirror'
  import Clipboard from 'clipboard'
  import { diGuiTree, searchTree } from './_source/resourceTree'

  let editor

  export default {
    name: 'shell',
    data () {
      return {
        valueConsistsOf: 'LEAF_PRIORITY',
        // script
        rawScript: '',
        // Custom parameter
        localParams: [],
        // resource(list)
        resourceList: [],
        // Cache ResourceList
        cacheResourceList: [],
        // define options
        options: [],
        normalizer (node) {
          return {
            label: node.name
          }
        },
        allNoResources: [],
        noRes: [],
        item: '',
        scriptBoxDialog: false
      }
    },
    mixins: [disabledState],
    props: {
      backfillItem: Object
    },
    methods: {
      _copyPath (e, node) {
        e.stopPropagation()
        let clipboard = new Clipboard('.copy-path', {
          text: function () {
            return node.raw.fullName
          }
        })
        clipboard.on('success', handler => {
          this.$message.success(`${i18n.$t('Copy success')}`)
          // Free memory
          clipboard.destroy()
        })
        clipboard.on('error', handler => {
          // Copy is not supported
          this.$message.warning(`${i18n.$t('The browser does not support automatic copying')}`)
          // Free memory
          clipboard.destroy()
        })
      },
      /**
       * return localParams
       */
      _onLocalParams (a) {
        this.localParams = a
      },
      setEditorVal () {
        this.item = editor.getValue()
        this.scriptBoxDialog = true
      },
      getSriptBoxValue (val) {
        editor.setValue(val)
        // this.scriptBoxDialog = false
      },
      closeAble () {
        // this.scriptBoxDialog = false
      },
      /**
       * return resourceList
       *
       */
      _onResourcesData (a) {
        this.resourceList = a
      },
      /**
       * cache resourceList
       */
      _onCacheResourcesData (a) {
        this.cacheResourceList = a
      },
      /**
       * verification
       */
      _verification () {
        // rawScript verification
        if (!editor.getValue()) {
          this.$message.warning(`${i18n.$t('Please enter script(required)')}`)
          return false
        }

        // localParams Subcomponent verification
        if (!this.$refs.refLocalParams._verifProp()) {
          return false
        }
        // noRes
        if (this.noRes.length > 0) {
          this.$message.warning(`${i18n.$t('Please delete all non-existent resources')}`)
          return false
        }
        // Process resourcelist
        let dataProcessing = _.map(this.resourceList, v => {
          return {
            id: v
          }
        })
        // storage
        this.$emit('on-params', {
          resourceList: dataProcessing,
          localParams: this.localParams,
          rawScript: editor.getValue()
        })
        return true
      },
      /**
       * Processing code highlighting
       */
      _handlerEditor () {
        // editor
        editor = codemirror('code-shell-mirror', {
          mode: 'shell',
          readOnly: this.isDetails
        })

        this.keypress = () => {
          if (!editor.getOption('readOnly')) {
            editor.showHint({
              completeSingle: false,
              extraKeys: {
                Enter: ''
              }
            })
          }
        }

        // Monitor keyboard
        editor.on('keypress', this.keypress)
        editor.setValue(this.rawScript)

        return editor
      },
      dataProcess (backResource) {
        let isResourceId = []
        let resourceIdArr = []
        if (this.resourceList.length > 0) {
          this.resourceList.forEach(v => {
            this.options.forEach(v1 => {
              if (searchTree(v1, v)) {
                isResourceId.push(searchTree(v1, v))
              }
            })
          })
          resourceIdArr = isResourceId.map(item => {
            return item.id
          })
          Array.prototype.diff = function (a) {
            return this.filter(function (i) { return a.indexOf(i) < 0 })
          }
          let diffSet = this.resourceList.diff(resourceIdArr)
          let optionsCmp = []
          if (diffSet.length > 0) {
            diffSet.forEach(item => {
              backResource.forEach(item1 => {
                if (item === item1.id || item === item1.res) {
                  optionsCmp.push(item1)
                }
              })
            })
          }
          let noResources = [{
            id: -1,
            name: $t('Unauthorized or deleted resources'),
            fullName: '/' + $t('Unauthorized or deleted resources'),
            children: []
          }]
          if (optionsCmp.length > 0) {
            this.allNoResources = optionsCmp
            optionsCmp = optionsCmp.map(item => {
              return { id: item.id, name: item.name, fullName: item.res }
            })
            optionsCmp.forEach(item => {
              item.isNew = true
            })
            noResources[0].children = optionsCmp
            this.options = this.options.concat(noResources)
          }
        }
      }
    },
    watch: {
      // Watch the cacheParams
      cacheParams (val) {
        this.$emit('on-cache-params', val)
      },
      resourceIdArr (arr) {
        let result = []
        arr.forEach(item => {
          this.allNoResources.forEach(item1 => {
            if (item.id === item1.id) {
              // resultBool = true
              result.push(item1)
            }
          })
        })
        this.noRes = result
      }
    },
    computed: {
      resourceIdArr () {
        let isResourceId = []
        let resourceIdArr = []
        if (this.resourceList.length > 0) {
          this.resourceList.forEach(v => {
            this.options.forEach(v1 => {
              if (searchTree(v1, v)) {
                isResourceId.push(searchTree(v1, v))
              }
            })
          })
          resourceIdArr = isResourceId.map(item => {
            return { id: item.id, name: item.name, res: item.fullName }
          })
        }
        return resourceIdArr
      },
      cacheParams () {
        return {
          resourceList: this.resourceIdArr,
          localParams: this.localParams
        }
      }
    },
    created () {
      let item = this.store.state.dag.resourcesListS
      diGuiTree(item)
      this.options = item
      let o = this.backfillItem

      // Non-null objects represent backfill
      if (!_.isEmpty(o)) {
        this.rawScript = o.params.rawScript || ''

        // backfill resourceList
        let backResource = o.params.resourceList || []
        let resourceList = o.params.resourceList || []
        if (resourceList.length) {
          _.map(resourceList, v => {
            if (!v.id) {
              this.store.dispatch('dag/getResourceId', {
                type: 'FILE',
                fullName: '/' + v.res
              }).then(res => {
                this.resourceList.push(res.id)
                this.dataProcess(backResource)
              }).catch(e => {
                this.resourceList.push(v.res)
                this.dataProcess(backResource)
              })
            } else {
              this.resourceList.push(v.id)
              this.dataProcess(backResource)
            }
          })
          this.cacheResourceList = resourceList
        }

        // backfill localParams
        let localParams = o.params.localParams || []
        if (localParams.length) {
          this.localParams = localParams
        }
      }
    },
    mounted () {
      setTimeout(() => {
        this._handlerEditor()
      }, 200)
    },
    destroyed () {
      if (editor) {
        editor.toTextArea() // Uninstall
        editor.off($('.code-shell-mirror'), 'keypress', this.keypress)
      }
    },
    components: { mLocalParams, mListBox, mScriptBox, Treeselect }
  }
</script>
