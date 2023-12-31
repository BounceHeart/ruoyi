<template>
  <div class="app-container">
    <el-form
      :model="queryParams"
      ref="queryForm"
      size="small"
      :inline="true"
      v-show="showSearch"
    >
      <el-form-item label="角色名称" prop="roleName">
        <el-input
          v-model="queryParams.roleName"
          placeholder="请输入角色名称"
          clearable
          style="width: 240px"
          @keyup.enter.native="handleQuery"
        />
      </el-form-item>
      <el-form-item label="权限字符" prop="roleKey">
        <el-input
          v-model="queryParams.roleKey"
          placeholder="请输入权限字符"
          clearable
          style="width: 240px"
          @keyup.enter.native="handleQuery"
        />
      </el-form-item>
      <el-form-item label="状态" prop="status">
        <el-select
          v-model="queryParams.status"
          placeholder="角色状态"
          clearable
          style="width: 240px"
        >
          <el-option
            v-for="dict in dict.type.sys_normal_disable"
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
      <right-toolbar
        :showSearch.sync="showSearch"
        @queryTable="getList"
      ></right-toolbar>
    </el-row>

    <el-table
      v-loading="loading"
      :data="roleList"
      @selection-change="handleSelectionChange"
    >
      <el-table-column type="selection" width="55" align="center" />
      <el-table-column label="角色编号" prop="roleId" width="120" />
      <el-table-column
        label="角色名称"
        prop="roleName"
        :show-overflow-tooltip="true"
        width="150"
      />
      <el-table-column
        label="权限字符"
        prop="roleKey"
        :show-overflow-tooltip="true"
        width="150"
      />
      <el-table-column label="显示顺序" prop="roleSort" width="100" />
      <el-table-column label="状态" align="center" width="100">
        <template slot-scope="scope">
          <el-switch
            v-model="scope.row.status"
            active-value="0"
            inactive-value="1"
            @change="handleStatusChange(scope.row)"
          ></el-switch>
        </template>
      </el-table-column>
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
        <template slot-scope="scope" v-if="scope.row.roleId !== 1">
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
          <el-dropdown
            size="mini"
            @command="(command) => handleCommand(command, scope.row)"
          >
            <el-button size="mini" type="text" icon="el-icon-d-arrow-right"
              >更多</el-button
            >
            <el-dropdown-menu slot="dropdown">
              <el-dropdown-item
                command="handleDataScope"
                icon="el-icon-circle-check"
                >数据权限</el-dropdown-item
              >
              <el-dropdown-item command="handleAuthUser" icon="el-icon-user"
                >分配用户</el-dropdown-item
              >
            </el-dropdown-menu>
          </el-dropdown>
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
      <el-form ref="form" :model="form" :rules="rules" label-width="100px">
        <el-form-item label="角色名称" prop="roleName">
          <el-input v-model="form.roleName" placeholder="请输入角色名称" />
        </el-form-item>
        <el-form-item prop="roleKey">
          <span slot="label">
            <el-tooltip
              content="控制器中定义的权限字符，如：@PreAuthorize(`@ss.hasRole('admin')`)"
              placement="top"
            >
              <i class="el-icon-question"></i>
            </el-tooltip>
            权限字符
          </span>
          <el-input v-model="form.roleKey" placeholder="请输入权限字符" />
        </el-form-item>
        <el-form-item label="角色顺序" prop="roleSort">
          <el-input-number
            v-model="form.roleSort"
            controls-position="right"
            :min="0"
          />
        </el-form-item>
        <el-form-item label="状态">
          <el-radio-group v-model="form.status">
            <el-radio
              v-for="dict in dict.type.sys_normal_disable"
              :key="dict.value"
              :label="dict.value"
              >{{ dict.label }}</el-radio
            >
          </el-radio-group>
        </el-form-item>
        <el-form-item label="菜单权限">
          <el-checkbox v-model="menuExpand">展开/折叠</el-checkbox>
          <el-checkbox v-model="menuNodeAll">全选/全不选</el-checkbox>
          <el-checkbox v-model="form.menuCheckStrictly">父子联动</el-checkbox>
          <el-tree
            class="tree-border"
            :data="menuOptions"
            show-checkbox
            ref="menu"
            node-key="id"
            :check-strictly="!form.menuCheckStrictly"
            empty-text="加载中，请稍候"
            :props="defaultProps"
          ></el-tree>
        </el-form-item>
        <el-form-item label="备注">
          <el-input
            v-model="form.remark"
            type="textarea"
            placeholder="请输入内容"
          ></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button type="primary" @click="submitForm">确 定</el-button>
        <el-button @click="cancel">取 消</el-button>
      </div>
    </el-dialog>

    <el-dialog
      :title="title"
      :visible.sync="openDataScope"
      width="500px"
      append-to-body
    >
      <el-form :model="form" label-width="80px">
        <el-form-item label="角色名称">
          <el-input v-model="form.roleName" :disabled="true" />
        </el-form-item>
        <el-form-item label="权限字符">
          <el-input v-model="form.roleKey" :disabled="true" />
        </el-form-item>
        <el-form-item label="权限范围">
          <el-select v-model="form.dataScope" @change="dataScopeSelectChange">
            <el-option
              v-for="item in dataScopeOptions"
              :key="item.value"
              :label="item.label"
              :value="item.value"
            ></el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="数据权限" v-show="form.dataScope == 2">
          <el-checkbox v-model="deptExpand">展开/折叠</el-checkbox>
          <el-checkbox v-model="deptNodeAll">全选/全不选</el-checkbox>
          <el-checkbox v-model="form.deptCheckStrictly">父子联动</el-checkbox>
          <el-tree
            class="tree-border"
            :data="deptOptions"
            show-checkbox
            default-expand-all
            ref="dept"
            node-key="id"
            :check-strictly="!form.deptCheckStrictly"
            empty-text="加载中，请稍候"
            :props="defaultProps"
          ></el-tree>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button type="primary" @click="submitDataScope">确 定</el-button>
        <el-button @click="cancelDataScope">取 消</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import {
  listRole,
  getRole,
  deptTreeSelect,
  addRole,
  updateRole,
  delRole,
  dataScope,
} from "../../../api/system/role";
import { treeselect, roleMenuTreeselect } from "../../../api/system/menu";
export default {
  name: "Role",
  dicts: ["sys_normal_disable"],
  data() {
    return {
      // 查询参数
      queryParams: {
        pageNum: 1,
        pageSize: 10,
        roleName: undefined,
        roleKey: undefined,
        status: undefined,
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
      // 角色表格数据
      roleList: [],
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
        roleName: [
          { required: true, message: "角色名称不能为空", trigger: "blur" },
        ],
        roleKey: [
          { required: true, message: "权限字符不能为空", trigger: "blur" },
        ],
        roleSort: [
          { required: true, message: "角色顺序不能为空", trigger: "blur" },
        ],
      },
      menuExpand: false,
      menuNodeAll: false,
      // 菜单列表
      menuOptions: [],
      defaultProps: {
        children: "children",
        label: "label",
      },
      // 是否显示弹出层（数据权限）
      openDataScope: false,
      deptExpand: true,
      deptNodeAll: false,
      // 数据范围选项
      dataScopeOptions: [
        {
          value: "1",
          label: "全部数据权限",
        },
        {
          value: "2",
          label: "自定数据权限",
        },
        {
          value: "3",
          label: "本部门数据权限",
        },
        {
          value: "4",
          label: "本部门及以下数据权限",
        },
        {
          value: "5",
          label: "仅本人数据权限",
        },
      ],
      // 部门列表
      deptOptions: [],
    };
  },
  created() {
    this.getList();
  },
  methods: {
    // 查询角色列表
    getList() {
      this.loading = true;
      setTimeout(() => {
        listRole(this.addDateRange(this.queryParams, this.dateRange))
          .then((res) => {
            this.roleList = res.rows;
            this.total = res.total;
          })
          .finally(() => {
            this.loading = false;
          });
      }, 1000);
    },
    //  查询菜单树结构
    getMenuTreeselect() {
      treeselect().then((res) => {
        this.menuOptions = res.data;
      });
    },
    // 根据角色ID查询菜单树结构
    getRoleMenuTreeselect(roleId) {
      roleMenuTreeselect(roleId).then((res) => {
        this.menuOptions = res.menus;
        let checkedKeys = res.checkedKeys;
        checkedKeys.forEach((v) => {
          this.$nextTick(() => {
            this.$refs.menu.setChecked(v, true, false);
          });
        });
      });
    },
    // 表单重置
    reset() {
      if (this.$refs.menu != undefined) {
        this.$refs.menu.setCheckedKeys([]);
      }
      (this.menuExpand = false),
        (this.menuNodeAll = false),
        (this.deptExpand = true),
        (this.deptNodeAll = false),
        (this.form = {
          roleId: undefined,
          roleName: undefined,
          roleKey: undefined,
          roleSort: 0,
          status: "0",
          menuIds: [],
          deptIds: [],
          menuCheckStrictly: true,
          deptCheckStrictly: true,
          remark: undefined,
        });
      this.resetForm("form");
    },
    // 多选框选中
    handleSelectionChange(selection) {
      this.ids = selection.map((item) => item.roleId);
      this.single = selection.length != 1;
      this.multiple = !selection.length;
    },
    // 搜索按钮
    handleQuery() {
      this.queryParams.pageNum = 1;
      this.getList();
    },
    // 重置按钮
    resetQuery() {
      this.resetForm("queryForm");
      this.dateRange = [];
      this.handleQuery();
    },
    // 新增按钮
    handleAdd() {
      this.reset();
      this.open = true;
      this.title = "添加角色";
      this.getMenuTreeselect();
    },
    // 修改按钮
    handleUpdate(row) {
      const roleId = row.roleId || this.ids;
      getRole(roleId).then((res) => {
        this.form = res.data;
        this.open = true;
        this.title = "修改角色";
      });
      this.getRoleMenuTreeselect(roleId);
    },
    // 删除按钮
    handleDelete(row) {
      const roleIds = row.roleId || this.ids;
      this.$confirm('是否确认删除角色编号为"' + roleIds + '"的数据项？', {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
        title: "系统提示",
      })
        .then(() => {
          delRole(roleIds);
          this.getList();
        })
        .catch(() => {});
    },
    // 提交按钮
    submitForm() {
      if (this.title == "添加角色") {
        addRole(this.form);
      } else if (this.title == "修改角色") {
        updateRole(this.form);
      }
      this.open = false;
      this.getList();
    },
    // 取消按钮
    cancel() {
      this.open = false;
      this.reset();
    },
    // 导出按钮
    handleExport() {
      this.download("system/role/export", { ...this.queryParams });
    },
    // 选择角色权限范围触发
    dataScopeSelectChange(value) {
      if (value !== "2") {
        this.$refs.dept.setCheckedKeys([]);
      }
    },
    // 更多操作触发
    handleCommand(command, row) {
      switch (command) {
        case "handleDataScope":
          this.handleDataScope(row);
          break;
        case "handleAuthUser":
          this.handleAuthUser(row);
          break;
        default:
          break;
      }
    },
    // 分配数据权限操作
    async handleDataScope(row) {
      this.reset();
      this.openDataScope = true;
      this.title = "分配数据权限";
      try {
        const res1 = await getRole(row.roleId);
        this.form = res1.data;
        const res2 = await deptTreeSelect(row.roleId);
        this.deptOptions = res2.depts;
        this.$refs.dept.setCheckedKeys(res2.checkedKeys);
      } catch (err) {}
    },
    // 提交按钮（数据权限）
    submitDataScope() {
      dataScope(this.form);
      this.openDataScope = false;
      this.getList();
    },
    // 取消按钮（数据权限）
    cancelDataScope() {
      this.openDataScope = false;
      this.reset();
    },
    /** 分配用户操作 */
    handleAuthUser(row) {
      const roleId = row.roleId;
      this.$router.push("authUser/" + roleId);
    },
  },
};
</script>

<style>
</style> 