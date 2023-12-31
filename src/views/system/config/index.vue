<template>
  <div class="app-container">
    <el-form
      :model="queryParams"
      ref="queryForm"
      size="small"
      :inline="true"
      v-show="showSearch"
      label-width="68px"
    >
      <el-form-item label="参数名称" prop="configName">
        <el-input
          v-model="queryParams.configName"
          placeholder="请输入参数名称"
          clearable
          style="width: 240px"
          @keyup.enter.native="handleQuery"
        />
      </el-form-item>
      <el-form-item label="参数键名" prop="configKey">
        <el-input
          v-model="queryParams.configKey"
          placeholder="请输入参数键名"
          clearable
          style="width: 240px"
          @keyup.enter.native="handleQuery"
        />
      </el-form-item>
      <el-form-item label="系统内置" prop="configType">
        <el-select
          v-model="queryParams.configType"
          placeholder="系统内置"
          clearable
          style="width: 240px"
        >
          <el-option
            v-for="dict in dict.type.sys_yes_no"
            :key="dict.value"
            :label="dict.label"
            :value="dict.value"
          />
        </el-select>
      </el-form-item>
      <el-form-item label="创建时间">
        <el-date-picker
          v-model="dateRange"
          style="width: 240px"
          value-format="yyyy-MM-dd"
          type="daterange"
          range-separator="-"
          start-placeholder="开始日期"
          end-placeholder="结束日期"
        ></el-date-picker>
      </el-form-item>
      <el-form-item>
        <el-button
          type="primary"
          icon="el-icon-search"
          size="mini"
          @click="handleQuery"
          >搜索</el-button
        >
        <el-button icon="el-icon-refresh" size="mini" @click="resetQuery"
          >重置</el-button
        >
      </el-form-item>
    </el-form>

    <el-row :gutter="10" class="mb8">
      <el-col :span="1.5">
        <el-button
          type="primary"
          plain
          icon="el-icon-plus"
          size="mini"
          @click="handleAdd"
          >新增</el-button
        >
      </el-col>
      <el-col :span="1.5">
        <el-button
          type="success"
          plain
          icon="el-icon-edit"
          size="mini"
          :disabled="single"
          @click="handleUpdate"
          >修改</el-button
        >
      </el-col>
      <el-col :span="1.5">
        <el-button
          type="danger"
          plain
          icon="el-icon-delete"
          size="mini"
          :disabled="multiple"
          @click="handleDelete"
          >删除</el-button
        >
      </el-col>
      <el-col :span="1.5">
        <el-button
          type="warning"
          plain
          icon="el-icon-download"
          size="mini"
          @click="handleExport"
          >导出</el-button
        >
      </el-col>
      <el-col :span="1.5">
        <el-button
          type="danger"
          plain
          icon="el-icon-refresh"
          size="mini"
          @click="handleRefreshCache"
          >刷新缓存</el-button
        >
      </el-col>
      <right-toolbar
        :showSearch.sync="showSearch"
        @queryTable="getList"
      ></right-toolbar>
    </el-row>

    <el-table
      v-loading="loading"
      :data="configList"
      @selection-change="handleSelectionChange"
    >
      <el-table-column type="selection" width="55" align="center" />
      <el-table-column label="参数主键" align="center" prop="configId" />
      <el-table-column
        label="参数名称"
        align="center"
        prop="configName"
        :show-overflow-tooltip="true"
      />
      <el-table-column
        label="参数键名"
        align="center"
        prop="configKey"
        :show-overflow-tooltip="true"
      />
      <el-table-column
        label="参数键值"
        align="center"
        prop="configValue"
        :show-overflow-tooltip="true"
      />
      <el-table-column label="系统内置" align="center" prop="configType">
        <template slot-scope="scope">
          <dict-tag
            :options="dict.type.sys_yes_no"
            :value="scope.row.configType"
          />
        </template>
      </el-table-column>
      <el-table-column
        label="备注"
        align="center"
        prop="remark"
        :show-overflow-tooltip="true"
      />
      <el-table-column
        label="创建时间"
        align="center"
        prop="createTime"
        width="180"
      >
        <template slot-scope="scope">
          <span>{{ parseTime(scope.row.createTime) }}</span>
        </template>
      </el-table-column>
      <el-table-column
        label="操作"
        align="center"
        class-name="small-padding fixed-width"
      >
        <template slot-scope="scope">
          <el-button
            size="mini"
            type="text"
            icon="el-icon-edit"
            @click="handleUpdate(scope.row)"
            >修改</el-button
          >
          <el-button
            size="mini"
            type="text"
            icon="el-icon-delete"
            @click="handleDelete(scope.row)"
            >删除</el-button
          >
        </template>
      </el-table-column>
    </el-table>

    <pagination
      v-show="total > 0"
      :total="total"
      :page.sync="queryParams.pageNum"
      :limit.sync="queryParams.pageSize"
      @pagination="getList"
    />

    <el-dialog :title="title" :visible.sync="open" width="500px" append-to-body>
      <el-form ref="form" :model="form" :rules="rules" label-width="80px">
        <el-form-item label="参数名称" prop="configName">
          <el-input v-model="form.configName" placeholder="请输入参数名称" />
        </el-form-item>
        <el-form-item label="参数键名" prop="configKey">
          <el-input v-model="form.configKey" placeholder="请输入参数键名" />
        </el-form-item>
        <el-form-item label="参数键值" prop="configValue">
          <el-input v-model="form.configValue" placeholder="请输入参数键值" />
        </el-form-item>
        <el-form-item label="系统内置" prop="configType">
          <el-radio-group v-model="form.configType">
            <el-radio
              v-for="dict in dict.type.sys_yes_no"
              :key="dict.value"
              :label="dict.value"
              >{{ dict.label }}</el-radio
            >
          </el-radio-group>
        </el-form-item>
        <el-form-item label="备注" prop="remark">
          <el-input
            v-model="form.remark"
            type="textarea"
            placeholder="请输入内容"
          />
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button type="primary" @click="submitForm">确 定</el-button>
        <el-button @click="cancel">取 消</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import {
  listConfig,
  getConfig,
  addConfig,
  updateConfig,
  delConfig,
  refreshCache,
} from "../../../api/system/config";
export default {
  name: "Config",
  dicts: ["sys_yes_no"],
  data() {
    return {
      // 查询参数
      queryParams: {
        pageNum: 1,
        pageSize: 10,
        configName: undefined,
        configKey: undefined,
        configType: undefined,
      },
      // 显示搜索条件
      showSearch: true,
      // 日期范围
      dateRange: [],
      // 选中数组
      ids: [],
      // 非单个禁用
      single: true,
      // 非多个禁用
      multiple: true,
      // 遮罩层
      loading: true,
      // 参数表格数据
      configList: [],
      // 总条数
      total: 0,
      // 弹出层标题
      title: "",
      // 是否显示弹出层
      open: false,
      // 表单参数
      form: {},
      // 表单校验
      rules: {
        configName: [
          { required: true, message: "参数名称不能为空", trigger: "blur" },
        ],
        configKey: [
          { required: true, message: "参数键名不能为空", trigger: "blur" },
        ],
        configValue: [
          { required: true, message: "参数键值不能为空", trigger: "blur" },
        ],
      },
    };
  },
  created() {
    this.getList();
  },
  methods: {
    // 查询参数列表
    getList() {
      this.loading = true;
      setTimeout(() => {
        listConfig(this.addDateRange(this.queryParams, this.dateRange))
          .then((res) => {
            this.configList = res.rows;
            this.total = res.total;
          })
          .finally(() => {
            this.loading = false;
          });
      }, 1000);
    },
    // 表单重置
    reset() {
      this.form = {
        configId: undefined,
        configName: undefined,
        configKey: undefined,
        configValue: undefined,
        configType: "Y",
        remark: undefined,
      };
      this.resetForm("form");
    },
    // 搜索按钮
    handleQuery() {
      this.queryParams.pageNum = 1;
      this.getList();
    },
    // 重置按钮
    resetQuery() {
      this.resetForm("queryForm");
      this.handleQuery();
    },
    // 新增按钮
    handleAdd() {
      this.reset();
      this.open = true;
      this.title = "添加参数";
    },
    // 修改按钮
    handleUpdate(row) {
      this.reset();
      this.open = true;
      this.title = "修改参数";
      const configId = row.configId || this.ids[0];
      getConfig(configId).then((res) => {
        this.form = res.data;
      });
    },
    // 删除按钮
    handleDelete(row) {
      const configId = row.configId || this.ids[0];
      this.$confirm("是否确认删除参数编号为'" + configId + "'的数据项?", {
        title: "系统提示",
        type: "warning",
      })
        .then(() => {
          delConfig(configId);
        })
        .catch(() => {
          this.$tip.message("已取消", "error");
        });
    },
    // 导出按钮
    handleExport() {
      this.download("/system/config/export", { ...this.queryParams });
    },
    // 刷新缓存按钮
    handleRefreshCache() {
      refreshCache()
        .then(() => {
          this.$tip.message("刷新成功", "success");
        })
        .catch(() => {
          this.$tip.message("刷新失败", "error");
        });
    },
    // 多选框选中数据
    handleSelectionChange(selection) {
      this.ids = selection.map((item) => item.configId);
      this.single = selection.length != 1;
      this.multiple = !selection.length;
    },
    // 提交按钮
    submitForm() {
      if (this.title == "添加参数") {
        addConfig(this.form);
      } else if (this.title == "修改参数") {
        updateConfig(this.form);
      }
      this.open = false;
      this.getList();
    },
    // 取消按钮
    cancel() {
      this.open = false;
      this.reset();
    },
  },
};
</script>

<style>
</style>