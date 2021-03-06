<template>
  <div class="wl-transfer" :style="{ width, height }">
    <template v-if="mode == 'transfer'">
      <!-- 左侧穿梭框 原料框 -->
      <div class="transfer-left">
        <h3 class="transfer-title">
          <el-checkbox
            :indeterminate="from_is_indeterminate"
            v-model="from_check_all"
            @change="fromAllBoxChange"
          ></el-checkbox>
          <span>{{ fromTitle }}</span>
          <slot name="title-left"></slot>
        </h3>
        <!-- 内容区 -->
        <div class="transfer-main">
          <slot name="from"></slot>
          <el-input
            v-if="filter"
            :placeholder="placeholder"
            v-model="filterFrom"
            size="small"
            class="filter-tree"
          ></el-input>
          <el-tree
            ref="from-tree"
            show-checkbox
            :lazy="lazy"
            :node-key="node_key"
            :load="leftloadNode"
            :props="defaultProps"
            :data="self_from_data"
            :default-expand-all="openAll"
            :highlight-current="highLight"
            :render-content="renderContentLeft"
            :filter-node-method="filterNodeFrom"
            :default-checked-keys="defaultCheckedKeys"
            :default-expanded-keys="from_expanded_keys"
            @check="fromTreeChecked"
          ></el-tree>
          <slot name="left-footer"></slot>
        </div>
      </div>
      <!-- 穿梭区 按钮框 -->
      <div class="transfer-center">
        <template v-if="button_text">
          <p class="transfer-center-item">
            <el-button
              type="primary"
              @click="addToAims(true)"
              :disabled="from_disabled"
            >
              {{ fromButton || "添加" }}
              <i class="el-icon-arrow-right"></i>
            </el-button>
          </p>
          <p class="transfer-center-item">
            <el-button
              type="primary"
              @click="removeToSource"
              :disabled="to_disabled"
              icon="el-icon-arrow-left"
              >{{ toButton || "移除" }}</el-button
            >
          </p>
        </template>
        <template v-else>
          <p class="transfer-center-item">
            <el-button
              type="primary"
              @click="addToAims(true)"
              icon="el-icon-arrow-right"
              circle
              :disabled="from_disabled"
            ></el-button>
          </p>
          <p class="transfer-center-item">
            <el-button
              type="primary"
              @click="removeToSource"
              :disabled="to_disabled"
              icon="el-icon-arrow-left"
              circle
            ></el-button>
          </p>
        </template>
      </div>
      <!-- 右侧穿梭框 目标框 -->
      <div class="transfer-right">
        <h3 class="transfer-title">
          <el-checkbox
            :indeterminate="to_is_indeterminate"
            v-model="to_check_all"
            @change="toAllBoxChange"
          ></el-checkbox>
          <span>{{ toTitle }}</span>
          <slot name="title-right"></slot>
        </h3>
        <!-- 内容区 -->
        <div class="transfer-main">
          <slot name="to"></slot>
          <el-input
            v-if="filter"
            :placeholder="placeholder"
            v-model="filterTo"
            size="small"
            class="filter-tree"
          ></el-input>
          <el-tree
            slot="to"
            ref="to-tree"
            show-checkbox
            :lazy="lazyRight"
            :data="self_to_data"
            :node-key="node_key"
            :props="defaultProps"
            :load="rightloadNode"
            :default-expand-all="openAll"
            :highlight-current="highLight"
            :render-content="renderContentRight"
            :filter-node-method="filterNodeTo"
            :default-expanded-keys="to_expanded_keys"
            @check="toTreeChecked"
          ></el-tree>
          <slot name="right-footer"></slot>
        </div>
      </div>
    </template>
    <template v-else-if="mode == 'addressList'">
      <!-- 左侧穿梭框 原料框 -->
      <div class="transfer-left">
        <h3 class="transfer-title">
          <el-checkbox
            :indeterminate="from_is_indeterminate"
            v-model="from_check_all"
            @change="fromAllBoxChange"
          ></el-checkbox>
          <span>{{ fromTitle }}</span>
        </h3>
        <!-- 内容区 -->
        <div class="transfer-main">
          <slot name="from"></slot>
          <el-input
            v-if="filter"
            :placeholder="placeholder"
            v-model="filterFrom"
            size="small"
            class="filter-tree"
          ></el-input>
          <el-tree
            ref="from-tree"
            show-checkbox
            :node-key="node_key"
            :props="defaultProps"
            :data="self_from_data"
            :default-expand-all="openAll"
            :highlight-current="highLight"
            :render-content="renderContentLeft"
            :filter-node-method="filterNodeFrom"
            :default-expanded-keys="from_expanded_keys"
            @check="fromTreeChecked"
          ></el-tree>
        </div>
      </div>
      <!-- 穿梭区 按钮框 -->
      <div class="transfer-center address-list-center">
        <p
          class="transfer-center-item"
          v-show="!move_up"
          :class="{ 'address-only-item': addressOptions.num === 1 }"
        >
          <el-button
            type="primary"
            @click="addressListTransfer(0)"
            icon="el-icon-arrow-right"
            circle
            class="address-first-btn"
            :disabled="from_disabled"
          ></el-button>
        </p>
        <p class="transfer-center-item" v-if="addressOptions.num > 1">
          <el-button
            type="primary"
            @click="addressListTransfer(1)"
            :disabled="from_disabled"
            icon="el-icon-arrow-right"
            circle
          ></el-button>
        </p>
        <p class="transfer-center-item" v-show="move_up">
          <el-button
            type="primary"
            @click="addressListTransfer(2)"
            :disabled="from_disabled"
            icon="el-icon-arrow-right"
            circle
          ></el-button>
        </p>
      </div>
      <div class="transfer-right">
        <div
          class="transfer-right-item"
          :class="{
            'transfer-right-small': move_up,
            'transfer-right-only': addressOptions.num === 1,
          }"
        >
          <h3 class="transfer-title">
            <span>{{ toTitle }}</span>
            <span class="u-clear" @click="clearList(0, 'all')" v-if="!move_up"
              >清空</span
            >
            <img
              class="move_up_img move_down_img"
              v-else
              :src="shang_img_src"
              alt="箭头图标"
              @click="moveUp('down')"
            />
          </h3>
          <!-- 内容区 -->
          <div class="transfer-main" v-if="!move_up">
            <slot name="to"></slot>
            <el-input
              v-if="filter"
              :placeholder="placeholder"
              v-model="filterListFirst"
              size="small"
              class="filter-tree"
            ></el-input>
            <ul class="address-list-ul">
              <li
                class="address-list-li"
                v-for="item of addressee"
                :key="item[node_key]"
              >
                <label>
                  {{ item[defaultProps.label] }}
                  {{ addressOptions.connector }}
                  {{ item[addressOptions.suffix] }}
                </label>
                <i
                  class="address-list-del el-icon-delete"
                  @click="clearList(0, item[node_key])"
                ></i>
              </li>
            </ul>
          </div>
        </div>
        <div class="transfer-right-item" v-if="addressOptions.num >= 2">
          <h3 class="transfer-title">
            <span>{{ toTitleSecond || "抄送人" }}</span>
            <span class="u-clear" @click="clearList(1, 'all')">清空</span>
          </h3>
          <!-- 内容区 -->
          <div class="transfer-main">
            <slot name="to"></slot>
            <el-input
              v-if="filter"
              :placeholder="placeholder"
              v-model="filterListSecond"
              size="small"
              class="filter-tree"
            ></el-input>
            <ul class="address-list-ul">
              <li
                class="address-list-li"
                v-for="item of Cc"
                :key="item[node_key]"
              >
                <label>
                  {{ item[defaultProps.label] }}
                  {{ addressOptions.connector }}
                  {{ item[addressOptions.suffix] }}
                </label>
                <i
                  class="address-list-del el-icon-delete"
                  @click="clearList(1, item[node_key])"
                ></i>
              </li>
            </ul>
          </div>
        </div>
        <div
          v-if="addressOptions.num === 3"
          class="transfer-right-item"
          :class="{ 'transfer-right-small': !move_up }"
        >
          <h3 class="transfer-title">
            <span>{{ toTitleThird || "密送人" }}</span>
            <span class="u-clear" @click="clearList(2, 'all')" v-if="move_up"
              >清空</span
            >
            <img
              class="move_up_img"
              v-else
              :src="shang_img_src"
              alt="箭头图标"
              @click="moveUp('up')"
            />
          </h3>
          <!-- 内容区 -->
          <div class="transfer-main" v-if="move_up">
            <slot name="to"></slot>
            <el-input
              v-if="filter"
              :placeholder="placeholder"
              v-model="filterListThird"
              size="small"
              class="filter-tree"
            ></el-input>
            <ul class="address-list-ul">
              <li
                class="address-list-li"
                v-for="item of secret_receiver"
                :key="item[node_key]"
              >
                <label>
                  {{ item[defaultProps.label] }}
                  {{ addressOptions.connector }}
                  {{ item[addressOptions.suffix] }}
                </label>
                <i
                  class="address-list-del el-icon-delete"
                  @click="clearList(2, item[node_key])"
                ></i>
              </li>
            </ul>
          </div>
        </div>
      </div>
    </template>
  </div>
</template>

<script>
import { arrayToTree, valInDeep } from "wl-core";
export default {
  name: "WlTransferTree",
  props: {
    sjr: {
      type: Array,
      default: () => {
        return [];
      },
    },
    csr: {
      type: Array,
      default: () => {
        return [];
      },
    },
    msr: {
      type: Array,
      default: () => {
        return [];
      },
    },
    // 宽度
    width: {
      type: String,
      default: "100%",
    },
    // 高度
    height: {
      type: String,
      default: "320px",
    },
    // 标题
    title: {
      type: Array,
      default: () => ["源列表", "目标列表"],
    },
    // 穿梭按钮名字
    button_text: Array,
    // 源数据
    from_data: {
      type: Array,
      default: () => [],
    },
    // 选中数据
    to_data: {
      type: Array,
      default: () => [],
    },
    // el-tree 配置项
    defaultProps: {
      type: Object,
      default: () => {
        return { label: "label", children: "children" };
      },
    },
    // el-tree node-key 必须唯一
    node_key: {
      type: String,
      default: "id",
    },
    // 自定义 pid参数名
    pid: {
      type: String,
      default: "pid",
    },
    // 自定义根节点pid的值，用于结束递归
    rootPidValue: {
      type: [String, Number],
      default: 0,
    },
    // 是否启用筛选
    filter: {
      type: Boolean,
      default: false,
    },
    // 是否展开所有节点
    openAll: {
      type: Boolean,
      default: false,
    },
    // 左侧自定义树节点
    renderContentLeft: Function,
    // 右侧自定义树节点
    renderContentRight: Function,
    // 穿梭框模式
    mode: {
      type: String,
      default: "transfer",
    },
    // 通讯录模式配置项 num-> 所需右侧通讯录个数 suffix-> label后想要拼接的字段（如id，即取此条数据的id拼接在后方）connector -> 连接符（字符串）
    addressOptions: {
      type: Object,
      default: () => {
        return {
          num: 3,
          suffix: "suffix",
          connector: "-",
        };
      },
    },
    // 穿梭后是否展开节点
    transferOpenNode: {
      type: Boolean,
      default: true,
    },
    // 源数据 默认选中节点
    defaultCheckedKeys: {
      type: Array,
      default: () => [],
    },
    // 源数据 默认展开节点
    defaultExpandedKeys: {
      type: Array,
      default: () => [],
    },
    // 筛选placeholder
    placeholder: {
      type: String,
      default: "输入关键字进行过滤",
    },
    // 自定义筛选函数
    filterNode: Function,
    // 默认穿梭一次默认选中数据
    defaultTransfer: {
      type: Boolean,
      default: false,
    },
    // 是否开启arrayToTree
    arrayToTree: {
      type: Boolean,
      default: false,
    },
    // 是否启用懒加载
    lazy: {
      type: Boolean,
      default: false,
    },
    // 是否右侧树也启用懒加载
    lazyRight: {
      type: Boolean,
      default: false,
    },
    // 懒加载的回调函数
    lazyFn: Function,
    // 是否高亮当前选中节点，默认值是 false。
    highLight: {
      type: Boolean,
      default: false,
    },
  },
  methods: {
    // -------------------------------提供输出函数---------------------
    // 添加按钮
    addToAims(emit) {
      // 获取选中通过穿梭框的keys - 仅用于传送纯净的id数组到父组件同后台通信
      let keys = this.$refs["from-tree"].getCheckedKeys();
      // 获取半选通过穿梭框的keys - 仅用于传送纯净的id数组到父组件同后台通信
      let harfKeys = this.$refs["from-tree"].getHalfCheckedKeys();
      // 选中节点数据
      let arrayCheckedNodes = this.$refs["from-tree"].getCheckedNodes();
      // 获取选中通过穿梭框的nodes - 仅用于传送选中节点数组到父组件同后台通信需求
      let nodes = JSON.parse(JSON.stringify(arrayCheckedNodes));
      // 半选中节点数据
      let arrayHalfCheckedNodes = this.$refs["from-tree"].getHalfCheckedNodes();
      // 获取半选通过穿梭框的nodes - 仅用于传送选中节点数组到父组件同后台通信需求
      let halfNodes = JSON.parse(JSON.stringify(arrayHalfCheckedNodes));

      // 自定义参数读取设置
      let children__ = this.defaultProps.children || "children";
      let pid__ = this.pid || "pid";
      let id__ = this["node_key"] || "id";
      let root__ = this.rootPidValue || 0;

      /*
       * 先整合目标树没有父节点的叶子节点选中，需要整理出来此叶子节点的父节点直到根节点路径 - 此时所有骨架节点已有
       * 再将所有末端叶子节点根据pid直接推入目标树即可
       * 声明新盒子将所有半选节点的子节点清除 - 只保留骨架 因为排序是先父后子 因此不存在子元素处理好插入时父元素还没处理的情况
       * 下面一二步是为了搭建出来目标树没有根节点躯干节点时的叶子选中，给此叶子搭建出根节点和躯干节点
       */

      // let不存在状态提升 因此在函数调用之前赋值 并递归为以为数组！
      // let self_to_data = JSON.stringify(this.self_to_data);
      // 第一步
      let skeletonHalfCheckedNodes = JSON.parse(
        JSON.stringify(arrayHalfCheckedNodes)
      ); // 深拷贝数据 - 半选节点
      // 筛选目标树不存在的骨架节点 - 半选内的节点
      let newSkeletonHalfCheckedNodes = [];
      skeletonHalfCheckedNodes.forEach((item) => {
        // 判断目标是否已在对面存在
        let inThere = valInDeep(
          this.self_to_data,
          item[id__],
          id__,
          children__
        );
        if (!inThere.length) {
          newSkeletonHalfCheckedNodes.push(item);
        }
      });
      // 筛选到目标树不存在的骨架后在处理每个骨架节点-非末端叶子节点 - 半选节点
      newSkeletonHalfCheckedNodes.forEach((item) => {
        item[children__] = [];
        root__ !== item[pid__]
          ? this.$refs["to-tree"].append(item, item[pid__])
          : this.$refs["to-tree"].append(item);
      });

      // 第二步
      // 筛选目标树不存在的骨架节点 - 全选内的节点
      let newSkeletonCheckedNodes = [];
      nodes.forEach((item) => {
        // 判断目标是否已在对面存在
        let inThere = valInDeep(
          this.self_to_data,
          item[id__],
          id__,
          children__
        );
        if (!inThere.length) {
          newSkeletonCheckedNodes.push(item);
        }
      });
      // 筛选到目标树不存在的骨架后在处理每个骨架节点-非末端叶子节点 - 全选节点
      newSkeletonCheckedNodes.forEach((item) => {
        if (item[children__] && item[children__].length > 0) {
          item[children__] = [];
          root__ !== item[pid__]
            ? this.$refs["to-tree"].append(item, item[pid__])
            : this.$refs["to-tree"].append(item);
        }
      });

      // 第三步 处理末端叶子元素 - 声明新盒子筛选出所有末端叶子节点
      let leafCheckedNodes = arrayCheckedNodes.filter(
        (item) => !item[children__] || item[children__].length == 0
      );
      // 末端叶子插入目标树
      leafCheckedNodes.forEach((item) => {
        // 判断目标是否已在对面存在
        let inThere = valInDeep(
          this.self_to_data,
          item[id__],
          id__,
          children__
        );
        if (!inThere.length) {
          this.$refs["to-tree"].append(item, item[pid__]);
        }
      });

      // 递归查询data内是否存在item函数
      /* function inquireIsExist(item, strData = self_to_data) {
        // 将树形数据格式化成一维字符串 然后通过匹配来判断是否已存在
        let strItem =
          typeof item[id__] == "number"
            ? `"${id__}":${item[id__]},`
            : `"${id__}":"${item[id__]}"`;
        let reg = RegExp(strItem);
        let existed = reg.test(strData);
        return existed;
      } */

      // 左侧删掉选中数据
      arrayCheckedNodes.map((item) => this.$refs["from-tree"].remove(item));

      // 处理完毕按钮恢复禁用状态
      this.from_check_keys = [];

      // 目标数据节点展开
      if (this.transferOpenNode && !this.lazy) {
        this.to_expanded_keys = keys;
      }

      // 传递信息给父组件
      emit &&
        this.$emit("addBtn", this.self_from_data, this.self_to_data, {
          keys,
          nodes,
          harfKeys,
          halfNodes,
        });

      // 处理完毕取消选中
      this.$refs["from-tree"].setCheckedKeys([]);
    },
    // 移除按钮
    removeToSource() {
      // 获取选中通过穿梭框的keys - 仅用于传送纯净的id数组到父组件同后台通信
      let keys = this.$refs["to-tree"].getCheckedKeys();
      // 获取半选通过穿梭框的keys - 仅用于传送纯净的id数组到父组件同后台通信
      let harfKeys = this.$refs["to-tree"].getHalfCheckedKeys();
      // 获取选中通过穿梭框的nodes 选中节点数据
      let arrayCheckedNodes = this.$refs["to-tree"].getCheckedNodes();
      // 获取选中通过穿梭框的nodes - 仅用于传送选中节点数组到父组件同后台通信需求
      let nodes = JSON.parse(JSON.stringify(arrayCheckedNodes));
      // 半选中节点数据
      let arrayHalfCheckedNodes = this.$refs["to-tree"].getHalfCheckedNodes();
      // 获取半选通过穿梭框的nodes - 仅用于传送选中节点数组到父组件同后台通信需求
      let halfNodes = JSON.parse(JSON.stringify(arrayHalfCheckedNodes));

      // 自定义参数读取设置
      let children__ = this.defaultProps.children || "children";
      let pid__ = this.pid || "pid";
      let id__ = this["node_key"] || "id";
      let root__ = this.rootPidValue || 0;

      /*
       * 先整合目标树没有父节点的叶子节点选中，需要整理出来此叶子节点的父节点直到根节点路径 - 此时所有骨架节点已有
       * 再将所有末端叶子节点根据pid直接推入目标树即可
       * 声明新盒子将所有半选节点的子节点清除 - 只保留骨架 因为排序是先父后子 因此不存在子元素处理好插入时父元素还没处理的情况
       * 下面一二步是为了搭建出来目标树没有根节点躯干节点时的叶子选中，给此叶子搭建出根节点和躯干节点
       */

      // let不存在状态提升 因此在函数调用之前赋值 并递归为以为数组！
      // let self_from_data = JSON.stringify(this.self_from_data);
      // 第一步
      let skeletonHalfCheckedNodes = JSON.parse(
        JSON.stringify(arrayHalfCheckedNodes)
      ); // 深拷贝数据 - 半选节点
      // 筛选目标树不存在的骨架节点 - 半选内的节点
      let newSkeletonHalfCheckedNodes = [];
      skeletonHalfCheckedNodes.forEach((item) => {
        // 判断目标是否已在对面存在
        let inThere = valInDeep(
          this.self_from_data,
          item[id__],
          id__,
          children__
        );
        if (!inThere.length) {
          newSkeletonHalfCheckedNodes.push(item);
        }
      });
      // 筛选到目标树不存在的骨架后在处理每个骨架节点-非末端叶子节点 - 半选节点
      newSkeletonHalfCheckedNodes.forEach((item) => {
        item[children__] = [];
        root__ !== item[pid__]
          ? this.$refs["from-tree"].append(item, item[pid__])
          : this.$refs["from-tree"].append(item);
      });

      // 第二步
      // 筛选目标树不存在的骨架节点 - 全选内的节点
      let newSkeletonCheckedNodes = [];
      nodes.forEach((item) => {
        // 判断目标是否已在对面存在
        let inThere = valInDeep(
          this.self_from_data,
          item[id__],
          id__,
          children__
        );
        if (!inThere.length) {
          newSkeletonCheckedNodes.push(item);
        }
      });
      // 筛选到目标树不存在的骨架后在处理每个骨架节点-非末端叶子节点 - 全选节点
      newSkeletonCheckedNodes.forEach((item) => {
        if (item[children__] && item[children__].length > 0) {
          item[children__] = [];
          root__ !== item[pid__]
            ? this.$refs["from-tree"].append(item, item[pid__])
            : this.$refs["from-tree"].append(item);
        }
      });

      // 第三步 处理末端叶子元素 - 声明新盒子筛选出所有末端叶子节点
      let leafCheckedNodes = arrayCheckedNodes.filter(
        (item) => !item[children__] || item[children__].length == 0
      );
      // 末端叶子插入目标树
      leafCheckedNodes.forEach((item) => {
        // 判断目标是否已在对面存在
        let inThere = valInDeep(
          this.self_from_data,
          item[id__],
          id__,
          children__
        );
        if (!inThere.length) {
          this.$refs["from-tree"].append(item, item[pid__]);
        }
      });

      // 递归查询data内是否存在item函数
      /* function inquireIsExist(item, strData = self_from_data) {
        // 将树形数据格式化成一维字符串 然后通过匹配来判断是否已存在
        let strItem =
          typeof item[id__] == "number"
            ? `"${id__}":${item[id__]},`
            : `"${id__}":"${item[id__]}"`;
        let reg = RegExp(strItem);
        let existed = reg.test(strData);
        return existed;
      } */

      // 右侧删掉选中数据
      arrayCheckedNodes.map((item) => this.$refs["to-tree"].remove(item));

      // 处理完毕按钮恢复禁用状态
      this.to_check_keys = [];

      // 目标数据节点展开
      if (this.transferOpenNode && !this.lazy) {
        this.from_expanded_keys = keys;
      }

      // 传递信息给父组件
      this.$emit("removeBtn", this.self_from_data, this.self_to_data, {
        keys,
        nodes,
        harfKeys,
        halfNodes,
      });
      // 处理完毕取消选中
      this.$refs["to-tree"].setCheckedKeys([]);
    },
    // 异步加载左侧
    leftloadNode(node, resolve) {
      this.lazyFn && this.lazyFn(node, resolve, "left");
    },
    // 异步加载右侧
    rightloadNode(node, resolve) {
      this.lazyFn && this.lazyFn(node, resolve, "right");
    },
    // 源树选中事件 - 是否禁用穿梭按钮
    fromTreeChecked(nodeObj, treeObj) {
      this.from_check_keys = treeObj.checkedNodes;
      this.$nextTick(() => {
        this.$emit("left-check-change", nodeObj, treeObj, this.from_check_all);
      });
    },
    // 目标树选中事件 - 是否禁用穿梭按钮
    toTreeChecked(nodeObj, treeObj) {
      this.to_check_keys = treeObj.checkedNodes;
      this.$nextTick(() => {
        this.$emit("right-check-change", nodeObj, treeObj, this.to_check_all);
      });
    },
    // 源数据 总全选checkbox
    fromAllBoxChange(val) {
      if (this.self_from_data.length == 0) {
        return;
      }
      if (val) {
        this.from_check_keys = this.self_from_data;
        this.$refs["from-tree"].setCheckedNodes(this.self_from_data);
      } else {
        this.$refs["from-tree"].setCheckedNodes([]);
        this.from_check_keys = [];
      }
      this.$emit("left-check-change", null, null, this.from_check_all);
    },
    // 目标数据 总全选checkbox
    toAllBoxChange(val) {
      if (this.self_to_data.length == 0) {
        return;
      }
      if (val) {
        this.to_check_keys = this.self_to_data;
        this.$refs["to-tree"].setCheckedNodes(this.self_to_data);
      } else {
        this.$refs["to-tree"].setCheckedNodes([]);
        this.to_check_keys = [];
      }
      this.$emit("right-check-change", null, null, this.to_check_all);
    },
    // 源数据 筛选
    filterNodeFrom(value, data) {
      if (this.filterNode) {
        return this.filterNode(value, data, "form");
      }
      if (!value) return true;
      return data[this.defaultProps.label].indexOf(value) !== -1;
    },
    // 目标数据筛选
    filterNodeTo(value, data) {
      if (this.filterNode) {
        return this.filterNode(value, data, "to");
      }
      if (!value) return true;
      return data[this.defaultProps.label].indexOf(value) !== -1;
    },
    // 通讯录模式 穿梭操作
    addressListTransfer(type) {
      // 获取选中通过穿梭框的keys - 仅用于传送纯净的id数组到父组件同后台通信
      let keys = this.$refs["from-tree"].getCheckedKeys(true);
      // 选中节点数据
      let arrayCheckedNodes = this.$refs["from-tree"].getCheckedNodes(true);
      // 去重筛选
      let arrayDeWeighting = [];
      switch (type) {
        case 0:
          arrayDeWeighting = arrayCheckedNodes.filter((item) => {
            if (
              !this.addressee.some(
                (ite) => ite[this.node_key] == item[this.node_key]
              )
            ) {
              return item;
            }
          });
          this.addressee = [...this.addressee, ...arrayDeWeighting];
          break;
        case 1:
          arrayDeWeighting = arrayCheckedNodes.filter((item) => {
            if (
              !this.Cc.some((ite) => ite[this.node_key] == item[this.node_key])
            ) {
              return item;
            }
          });
          this.Cc = [...this.Cc, ...arrayDeWeighting];
          break;
        case 2:
          arrayDeWeighting = arrayCheckedNodes.filter((item) => {
            if (
              !this.secret_receiver.some(
                (ite) => ite[this.node_key] == item[this.node_key]
              )
            ) {
              return item;
            }
          });
          this.secret_receiver = [...this.secret_receiver, ...arrayDeWeighting];
          break;
      }

      // 处理完毕取消选中
      this.$refs["from-tree"].setCheckedKeys([]);

      // 处理完毕按钮恢复禁用状态
      this.from_check_keys = [];

      // 传递信息给父组件
      this.$emit("addBtn", this.addressee, this.Cc, this.secret_receiver);
    },
    // 清理 通讯录选中 数据
    clearList(type, id) {
      switch (type) {
        case 0:
          this.addressee =
            id == "all"
              ? []
              : this.addressee.filter((item) => item[this.node_key] != id);
          break;
        case 1:
          this.Cc =
            id == "all"
              ? []
              : this.Cc.filter((item) => item[this.node_key] != id);
          break;
        case 2:
          this.secret_receiver =
            id == "all"
              ? []
              : this.secret_receiver.filter(
                  (item) => item[this.node_key] != id
                );
          break;
      }
      // 传递信息给父组件
      this.$emit("removeBtn", this.addressee, this.Cc, this.secret_receiver);
    },
    // 右侧 通讯录 上下自动
    moveUp(type) {
      if (type == "up") {
        this.move_up = true;
      } else {
        this.move_up = false;
      }
    },
    // 以下为提供方法 ----------------------------------------------------------------方法--------------------------------------
    /**
     * @name 清空选中节点
     * @param {String} type left左边 right右边 all全部 默认all
     */
    clearChecked(type = "all") {
      if (type === "left") {
        this.$refs["from-tree"].setCheckedKeys([]);
        this.from_is_indeterminate = false;
        this.from_check_all = false;
      } else if (type === "right") {
        this.$refs["to-tree"].setCheckedKeys([]);
        this.to_is_indeterminate = false;
        this.to_check_all = false;
      } else {
        this.$refs["from-tree"].setCheckedKeys([]);
        this.$refs["to-tree"].setCheckedKeys([]);
        this.from_is_indeterminate = false;
        this.from_check_all = false;
        this.to_is_indeterminate = false;
        this.to_check_all = false;
      }
    },
    /**
     * @name 获取选中数据
     */
    getChecked() {
      // 左侧选中信息
      let leftKeys = this.$refs["from-tree"].getCheckedKeys();
      let leftHarfKeys = this.$refs["from-tree"].getHalfCheckedKeys();
      let leftNodes = this.$refs["from-tree"].getCheckedNodes();
      let leftHalfNodes = this.$refs["from-tree"].getHalfCheckedNodes();
      // 右侧选中信息
      let rightKeys = this.$refs["to-tree"].getCheckedKeys();
      let rightHarfKeys = this.$refs["to-tree"].getHalfCheckedKeys();
      let rightNodes = this.$refs["to-tree"].getCheckedNodes();
      let rightHalfNodes = this.$refs["to-tree"].getHalfCheckedNodes();
      return {
        leftKeys,
        leftHarfKeys,
        leftNodes,
        leftHalfNodes,
        rightKeys,
        rightHarfKeys,
        rightNodes,
        rightHalfNodes,
      };
    },
    /**
     * @name 设置选中数据
     * @param {Array} leftKeys 左侧ids
     * @param {Array} rightKeys 右侧ids
     */
    setChecked(leftKeys = [], rightKeys = []) {
      this.$refs["from-tree"].setCheckedKeys(leftKeys);
      this.$refs["to-tree"].setCheckedKeys(rightKeys);
    },
  },
  computed: {
    // 左侧数据
    self_from_data() {
      let from_array = [...this.from_data];
      return !this.arrayToTree
        ? from_array
        : arrayToTree(from_array, {
            id: this.node_key,
            pid: this.pid,
            children: this.defaultProps.children,
          });
    },
    // 右侧数据
    self_to_data() {
      let to_array = [...this.to_data];
      return !this.arrayToTree
        ? to_array
        : arrayToTree(to_array, {
            id: this.node_key,
            pid: this.pid,
            children: this.defaultProps.children,
          });
    },
    // 左侧菜单名
    fromTitle() {
      let [text] = this.title;
      return text;
    },
    // 右侧菜单名
    toTitle() {
      let [, text] = this.title;
      return text;
    },
    // 右侧菜单名2
    toTitleSecond() {
      let [, , text] = this.title;
      return text;
    },
    // 右侧菜单名3
    toTitleThird() {
      let [, , , text] = this.title;
      return text;
    },
    // 上部按钮名
    fromButton() {
      if (this.button_text == undefined) {
        return;
      }

      let [text] = this.button_text;
      return text;
    },
    // 下部按钮名
    toButton() {
      if (this.button_text == undefined) {
        return;
      }
      let [, text] = this.button_text;
      return text;
    },
  },
  watch: {
    // 左侧 状态监测
    from_check_keys(val) {
      if (val.length > 0) {
        // 穿梭按钮是否禁用
        this.from_disabled = false;
        // 总半选是否开启
        this.from_is_indeterminate = true;

        // 总全选是否开启 - 根据选中节点中为根节点的数量是否和源数据长度相等
        let allCheck = val.filter((item) => item[this.pid] == 0);
        if (allCheck.length == this.self_from_data.length) {
          // 关闭半选 开启全选
          this.from_is_indeterminate = false;
          this.from_check_all = true;
        } else {
          this.from_is_indeterminate = true;
          this.from_check_all = false;
        }
      } else {
        this.from_disabled = true;
        this.from_is_indeterminate = false;
        this.from_check_all = false;
      }
    },
    // 右侧 状态监测
    to_check_keys(val) {
      if (val.length > 0) {
        // 穿梭按钮是否禁用
        this.to_disabled = false;
        // 总半选是否开启
        this.to_is_indeterminate = true;

        // 总全选是否开启 - 根据选中节点中为根节点的数量是否和源数据长度相等
        let allCheck = val.filter((item) => item[this.pid] == 0);
        if (allCheck.length == this.self_to_data.length) {
          // 关闭半选 开启全选
          this.to_is_indeterminate = false;
          this.to_check_all = true;
        } else {
          this.to_is_indeterminate = true;
          this.to_check_all = false;
        }
      } else {
        this.to_disabled = true;
        this.to_is_indeterminate = false;
        this.to_check_all = false;
      }
    },
    // 左侧 数据筛选
    filterFrom(val) {
      this.$refs["from-tree"].filter(val);
    },
    // 右侧 数据筛选
    filterTo(val) {
      this.$refs["to-tree"].filter(val);
    },
    // 通讯录模式 右1筛选
    filterListFirst(newval, oldval) {
      if (oldval == "") {
        this.archiveFirst = this.addressee;
      }
      if (newval == "") {
        this.addressee = this.archiveFirst;
      }
      let reg = RegExp(newval);
      this.addressee = this.addressee.filter((item) => reg.test(item.label));
    },
    // 通讯录模式 右2筛选
    filterListSecond(newval, oldval) {
      if (oldval == "") {
        this.archiveSecond = this.Cc;
      }
      if (newval == "") {
        this.Cc = this.archiveSecond;
      }
      let reg = RegExp(newval);
      this.Cc = this.Cc.filter((item) => reg.test(item.label));
    },
    // 通讯录模式 右3筛选
    filterListThird(newval, oldval) {
      if (oldval == "") {
        this.archiveThird = this.secret_receiver;
      }
      if (newval == "") {
        this.secret_receiver = this.archiveThird;
      }
      let reg = RegExp(newval);
      this.secret_receiver = this.secret_receiver.filter((item) =>
        reg.test(item.label)
      );
    },
    // 监视默认选中
    defaultCheckedKeys: {
      handler(val) {
        this.from_check_keys = val || [];
        if (this.defaultTransfer && this.from_check_keys.length) {
          this.$nextTick(() => {
            this.addToAims(false);
          });
        }
      },
      immediate: true,
    },
    // 监视默认展开
    defaultExpandedKeys: {
      handler(val) {
        let _form = new Set(this.from_expanded_keys.concat(val));
        this.from_expanded_keys = [..._form];
        let _to = new Set(this.to_expanded_keys.concat(val));
        this.to_expanded_keys = [..._to];
      },
      immediate: true,
    },
    // 收件人默认值监测
    sjr: {
      handler(val) {
        this.addressee.push(...val);
      },
      immediate: true,
    },
    // 抄送人默认值监测
    csr: {
      handler(val) {
        this.Cc.push(...val);
      },
      immediate: true,
    },
    // 密送人默认值监测
    msr: {
      handler(val) {
        this.secret_receiver.push(...val);
      },
      immediate: true,
    },
  },
};
</script>

<style scoped>
@import "~@/assets/css/base/clear.min.css";
@import "~@/assets/css/transfer-tree/index.min.css";
</style>
