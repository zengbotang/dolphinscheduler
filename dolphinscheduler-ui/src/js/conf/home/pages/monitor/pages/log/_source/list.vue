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
  <div class="list-model">
    <div class="table-box">
      <el-table :data="list" size="mini" style="width: 100%">
        <el-table-column type="index" :label="$t('#')" width="50"></el-table-column>
        <el-table-column prop="userName" :label="$t('User Name')"></el-table-column>
        <el-table-column prop="resource" :label="$t('Resource Type')"></el-table-column>
        <el-table-column prop="resourceName" :label="$t('Project Name')"></el-table-column>
        <el-table-column prop="operation" :label="$t('Operation')">
          <template slot-scope="scope">
            <span>{{scope.row.operation | filterNull}}</span>
          </template>
        </el-table-column>
        <el-table-column :label="$t('Create Time')" min-width="120">
          <template slot-scope="scope">
            <span>{{scope.row.time | formatDate}}</span>
          </template>
        </el-table-column>
      </el-table>
    </div>
  </div>
</template>
<script>
  export default {
    name: 'audit-list',
    data () {
      return {
        list: []
      }
    },
    props: {
      logList: Array,
      pageNo: Number,
      pageSize: Number
    },
    methods: {
      _edit (item) {
        this.$emit('on-edit', item)
      }
    },
    watch: {
      logList (a) {
        this.list = []
        setTimeout(() => {
          this.list = a
        })
      }
    },
    created () {
      this.list = this.logList
    },
    mounted () {
    },
    components: { }
  }
</script>
