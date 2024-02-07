# 前端解决树形结构方案 UniApplication 

## 模版源代码

~~~vue
<!-- 树形层级选择器-->
<!-- 1、支持单选、多选 -->
<template>
	<view>
		<view class="tree-cover" :class="{'show':showDialog}" @tap="_cancel"></view>
		<view class="tree-dialog" :class="{'show':showDialog}">
			<view class="tree-bar">
				<view class="tree-bar-cancel" :style="{'color':cancelColor}" hover-class="hover-c" @tap="_cancel">取消
				</view>
				<view class="tree-bar-title" :style="{'color':titleColor}">{{title}}</view>
                <view class="tree-bar-cancel" style="color:blue;margin-left:150px" hover-class="hover-c" @tap="_submit">确定
				</view>
				<view class="tree-bar-confirm" :style="{'color':confirmColor}" hover-class="hover-c" @tap="_confirm">
					{{multiple?'确定':''}}
				</view>
			</view>
			<view class="tree-view">
				<scroll-view class="tree-list" :scroll-y="true">
					<block v-for="(item, index) in treeList" :key="index">
						<view class="tree-item" :style="[{
							paddingLeft: item.level*30 + 'rpx'
						}]" :class="{
							itemBorder: border === true,
							show: item.isShow
						}">
							<view class="item-label">
								<view class="item-icon uni-inline-item" @tap.stop="_onItemSwitch(item, index)">
									<view v-if="!item.isLastLevel&&item.isShowChild" class="switch-on"
										:style="{'border-left-color':switchColor}">
									</view>
									<view v-else-if="!item.isLastLevel&&!item.isShowChild" class="switch-off"
										:style="{'border-top-color':switchColor}">
									</view>
									<view v-else class="item-last-dot" :style="{'border-top-color':switchColor}">
									</view>
								</view>
								<view class="uni-flex-item item-cont uni-inline-item" @tap.stop="_onItemSelect(item, index)">
									<view class="item-name"> {{item.name+(item.childCount?"("+item.childCount+")":'')}}
									</view>
									<view class="item-check" v-if="selectParent?true:item.isLastLevel">
										<view class="item-check-yes" v-if="item.checkStatus==1"
											:class="{'radio':!multiple}" :style="{'border-color':confirmColor}">
											<view class="item-check-yes-part"
												:style="{'background-color':confirmColor}">
											</view>
										</view>
										<view class="item-check-yes" v-else-if="item.checkStatus==2"
											:class="{'radio':!multiple}" :style="{'border-color':confirmColor}">
											<view class="item-check-yes-all" :style="{'background-color':confirmColor}">
											</view>
										</view>
										<view class="item-check-no" v-else :class="{'radio':!multiple}"
											:style="{'border-color':confirmColor}"></view>
									</view>
								</view>
							</view>

						</view>
					</block>
				</scroll-view>
			</view>
		</view>
	</view>
</template>

<script>
	export default {
		emits: ['select-change'],
		name: "ba-tree-picker",
		props: {
			showDialog: {
				type: Boolean,
				default: false
			},
			valueKey: {
				type: String,
				default: 'id'
			},
			textKey: {
				type: String,
				default: 'name'
			},
			childrenKey: {
				type: String,
				default: 'children'
			},
			localdata: {
				type: Array,
				default: function() {
					return []
				}
			},
			localTreeList: { //在已经格式化好的数据
				type: Array,
				default: function() {
					return []
				}
			},
			selectedData: {
				type: Array,
				default: function() {
					return []
				}
			},
			title: {
				type: String,
				default: ''
			},
			multiple: { // 是否可以多选
				type: Boolean,
				default: true
			},
			selectParent: { //是否可以选父级
				type: Boolean,
				default: true
			},
			confirmColor: { // 确定按钮颜色
				type: String,
				default: '' // #30bf78
			},
			cancelColor: { // 取消按钮颜色
				type: String,
				default: '' // #757575
			},
			titleColor: { // 标题颜色
				type: String,
				default: '' //
			},
			switchColor: { // 节点切换图标颜色
				type: String,
				default: '' // #666
			},
			border: { // 是否有分割线
				type: Boolean,
				default: false
			},
		},
		data() {
			return {
				// showDialog: false,
				treeList: []
			}
		},
		computed: {},
		methods: {
			_show() {
				// this.showDialog = true
			},
			_hide() {
				this.$emit("cancel", '');

				// this.showDialog = false
			},
			_cancel() {
				// this._hide()
				this.$emit("cancel", '');
			},
            _submit(){
                this.$emit("submit", '');
            },
			_confirm() { //多选
				let selectedList = []; //如果子集全部选中，只返回父级 id
				let selectedNames;
				let currentLevel = -1;
				let selectItem=[]
				this.treeList.forEach((item, index) => {
					if (currentLevel >= 0 && item.level > currentLevel) {

					} else {
						if (item.checkStatus === 2) {
							currentLevel = item.level;
							selectedList.push(item.id);
							selectItem.push(selectItem)
							selectedNames = selectedNames ? selectedNames + ' / ' + item.name : item.name;
						} else {
							currentLevel = -1;
						}
					}
				})
				//console.log('_confirm', selectedList);
				// this._hide()
				this.$emit("select-change",selectItem, selectedList, selectedNames);
			},
			//格式化原数据（原数据为tree结构）
			_formatTreeData(list = [], level = 0, parentItem, isShowChild = true) {
				let nextIndex = 0;
				let parentId = -1;
				let initCheckStatus = 0;
				if (parentItem) {
					nextIndex = this.treeList.findIndex(item => item.id === parentItem.id) + 1;
					parentId = parentItem.id;
					if (!this.multiple) { //单选
						initCheckStatus = 0;
					} else
						initCheckStatus = parentItem.checkStatus == 2 ? 2 : 0;
				}
				list.forEach(item => {
					let isLastLevel = true;
					if (item && item[this.childrenKey]) {
						let children = item[this.childrenKey];
						if (Array.isArray(children) && children.length > 0) {
							isLastLevel = false;
						}
					}

					let itemT = {
						id: item[this.valueKey],
						name: item[this.textKey],
						level,
						isLastLevel,
						isShow: isShowChild,
						isShowChild: false,
						checkStatus: initCheckStatus,
						orCheckStatus: 0,
						parentId,
						children: item[this.childrenKey],
						childCount: item[this.childrenKey] ? item[this.childrenKey].length : 0,
						childCheckCount: 0,
						childCheckPCount: 0,
						resetItem:item
					};

					if (this.selectedData.indexOf(itemT.id) >= 0) {
						itemT.checkStatus = 2;
						itemT.orCheckStatus = 2;
						itemT.childCheckCount = itemT.children ? itemT.children.length : 0;
						this._onItemParentSelect(itemT, nextIndex);
					}

					this.treeList.splice(nextIndex, 0, itemT);
					nextIndex++;
				})
				//console.log(this.treeList);
			},
			// 节点打开、关闭切换
			_onItemSwitch(item, index) {
				// console.log(item)
				//console.log('_itemSwitch')
				if (item.isLastLevel === true) {
					return;
				}
				item.isShowChild = !item.isShowChild;
				if (item.children) {
					this._formatTreeData(item.children, item.level + 1, item);
					item.children = undefined;
				} else {
					this._onItemChildSwitch(item, index);
				}
			},
			_onItemChildSwitch(item, index) {
				//console.log('_onItemChildSwitch')
				const firstChildIndex = index + 1;
				if (firstChildIndex > 0)
					for (var i = firstChildIndex; i < this.treeList.length; i++) {
						let itemChild = this.treeList[i];
						if (itemChild.level > item.level) {
							if (item.isShowChild) {
								if (itemChild.parentId === item.id) {
									itemChild.isShow = item.isShowChild;
									if (!itemChild.isShow) {
										itemChild.isShowChild = false;
									}
								}
							} else {
								itemChild.isShow = item.isShowChild;
								itemChild.isShowChild = false;
							}
						} else {
							return;
						}
					}
			},
			// 节点选中、取消选中
			_onItemSelect(item, index) {
				//console.log('_onItemSelect')
				//console.log(item)
				if (!this.multiple) { //单选
					item.checkStatus = item.checkStatus == 0 ? 2 : 0;

					this.treeList.forEach((v, i) => {
						if (i != index) {
							this.treeList[i].checkStatus = 0
						} else {
							this.treeList[i].checkStatus = 2
						}
					})

					let selectedList = [];
					let selectedNames;
					let selectItem=[]
					selectedList.push(item.id);
					selectItem.push(item)
					selectedNames = item.name;
					// this._hide()
					this.$emit("select-change",selectItem, selectedList, selectedNames);
					return
				}

				let oldCheckStatus = item.checkStatus;
				switch (oldCheckStatus) {
					case 0:
						item.checkStatus = 2;
						item.childCheckCount = item.childCount;
						item.childCheckPCount = 0;
						break;
					case 1:
					case 2:
						item.checkStatus = 0;
						item.childCheckCount = 0;
						item.childCheckPCount = 0;
						break;
					default:
						break;
				}
				//子节点 全部选中
				this._onItemChildSelect(item, index);
				//父节点 选中状态变化
				this._onItemParentSelect(item, index, oldCheckStatus);
			},
			_onItemChildSelect(item, index) {
				//console.log('_onItemChildSelect')
				let allChildCount = 0;
				if (item.childCount && item.childCount > 0) {
					index++;
					while (index < this.treeList.length && this.treeList[index].level > item.level) {
						let itemChild = this.treeList[index];
						itemChild.checkStatus = item.checkStatus;
						if (itemChild.checkStatus == 2) {
							itemChild.childCheckCount = itemChild.childCount;
							itemChild.childCheckPCount = 0;
						} else if (itemChild.checkStatus == 0) {
							itemChild.childCheckCount = 0;
							itemChild.childCheckPCount = 0;
						}
						// console.log('>>>>index：', index, 'item：', itemChild.name, '  status：', itemChild
						// 	.checkStatus)
						index++;
					}
				}
			},
			_onItemParentSelect(item, index, oldCheckStatus) {
				//console.log('_onItemParentSelect')
				//console.log(item)
				const parentIndex = this.treeList.findIndex(itemP => itemP.id == item.parentId);
				//console.log('parentIndex：' + parentIndex)
				if (parentIndex >= 0) {
					let itemParent = this.treeList[parentIndex];
					let count = itemParent.childCheckCount;
					let oldCheckStatusParent = itemParent.checkStatus;

					if (oldCheckStatus == 1) {
						itemParent.childCheckPCount -= 1;
					} else if (oldCheckStatus == 2) {
						itemParent.childCheckCount -= 1;
					}
					if (item.checkStatus == 1) {
						itemParent.childCheckPCount += 1;
					} else if (item.checkStatus == 2) {
						itemParent.childCheckCount += 1;
					}

					if (itemParent.childCheckCount <= 0 && itemParent.childCheckPCount <= 0) {
						itemParent.childCheckCount = 0;
						itemParent.childCheckPCount = 0;
						itemParent.checkStatus = 0;
					} else if (itemParent.childCheckCount >= itemParent.childCount) {
						itemParent.childCheckCount = itemParent.childCount;
						itemParent.childCheckPCount = 0;
						itemParent.checkStatus = 2;
					} else {
						itemParent.checkStatus = 1;
					}
					//console.log('itemParent：', itemParent)
					this._onItemParentSelect(itemParent, parentIndex, oldCheckStatusParent);
				}
			},
			// 重置数据
			_reTreeList() {
				this.treeList.forEach((v, i) => {
					this.treeList[i].checkStatus = v.orCheckStatus
				})
			},
			_initTree() {
				this.treeList = [];
				this._formatTreeData(this.localdata);
			}
		},
		watch: {
			localdata() {
				this._initTree();
			},
			localTreeList() {
				this.treeList = this.localTreeList;
			}
		},
		mounted() {
			this._initTree();
		}
	}
</script>

<style scoped>
	.tree-cover {
		position: fixed;
		top: 0rpx;
		right: 0rpx;
		bottom: 0rpx;
		left: 0rpx;
		z-index: 100;
		background-color: rgba(0, 0, 0, .4);
		opacity: 0;
		transition: all 0.3s ease;
		visibility: hidden;
	}

	.tree-cover.show {
		visibility: visible;
		opacity: 1;
	}

	.tree-dialog {
		position: fixed;
		top: 0rpx;
		right: 0rpx;
		bottom: 0rpx;
		left: 0rpx;
		background-color: #fff;
		border-top-left-radius: 10px;
		border-top-right-radius: 10px;
		/* #ifndef APP-NVUE */
		display: flex;
		/* #endif */
		flex-direction: column;
		z-index: 102;
		top: 20%;
		transition: all 0.3s ease;
		transform: translateY(100%);
	}

	.tree-dialog.show {
		transform: translateY(0);
	}

	.tree-bar {
		/* background-color: #fff; */
		height: 90rpx;
		padding-left: 25rpx;
		padding-right: 25rpx;
		display: flex;
		justify-content: space-between;
		align-items: center;
		box-sizing: border-box;
		border-bottom-width: 1rpx !important;
		border-bottom-style: solid;
		border-bottom-color: #f5f5f5;
		font-size: 32rpx;
		color: #757575;
		line-height: 1;
	}

	.tree-bar-confirm {
		color: #30bf78;
		padding: 15rpx;
	}

	.tree-bar-title {}

	.tree-bar-cancel {
		color: #757575;
		padding: 15rpx;
	}

	.tree-view {
		flex: 1;
		padding: 20rpx;
		/* #ifndef APP-NVUE */
		display: flex;
		/* #endif */
		flex-direction: column;
		overflow: hidden;
		height: 100%;
	}

	.tree-list {
		flex: 1;
		height: 100%;
		overflow: hidden;
	}

	.tree-item {
		display: flex;
		justify-content: space-between;
		align-items: center;
		line-height: 1;
		height: 0;
		opacity: 0;
		transition: 0.2s;
		overflow: hidden;
	}

	.tree-item.show {
		height: 90rpx;
		opacity: 1;
	}

	.tree-item.showchild:before {
		transform: rotate(90deg);
	}

	.tree-item.last:before {
		opacity: 0;
	}

	.switch-on {
		width: 0;
		height: 0;
		border-left: 10rpx solid transparent;
		border-right: 10rpx solid transparent;
		border-top: 15rpx solid #666;
	}

	.switch-off {
		width: 0;
		height: 0;
		border-bottom: 10rpx solid transparent;
		border-top: 10rpx solid transparent;
		border-left: 15rpx solid #666;
	}

	.item-last-dot {
		position: absolute;
		width: 10rpx;
		height: 10rpx;
		border-radius: 100%;
		background: #666;
	}

	.item-icon {
		width: 26rpx;
		height: 24rpx;
		margin-right: 8rpx;
		padding-right: 20rpx;
		padding-left: 20rpx;
	}

	.item-label {
		flex: 1;
		display: flex;
		align-items: center;
		height: 100%;
		line-height: 1.2;
	}
.item-cont{
	display: flex;
	justify-content: space-between;
	align-items: center;
	/* 选中按钮排列到右侧 */
	width: 100%; 
}
	.item-name {
		flex: 1;
		overflow: hidden;
		text-overflow: ellipsis;
		white-space: nowrap;
		width: 450rpx;
	}

	.item-check {
		width: 40px;
		height: 40px;
		display: flex;
		justify-content: center;
		align-items: center;
	}

	.item-check-yes,
	.item-check-no {
		width: 20px;
		height: 20px;
		border-top-left-radius: 20%;
		border-top-right-radius: 20%;
		border-bottom-right-radius: 20%;
		border-bottom-left-radius: 20%;
		border-top-width: 1rpx;
		border-left-width: 1rpx;
		border-bottom-width: 1rpx;
		border-right-width: 1rpx;
		border-style: solid;
		border-color: #30bf78;
		display: flex;
		justify-content: center;
		align-items: center;
		box-sizing: border-box;
	}

	.item-check-yes-part {
		width: 12px;
		height: 12px;
		border-top-left-radius: 20%;
		border-top-right-radius: 20%;
		border-bottom-right-radius: 20%;
		border-bottom-left-radius: 20%;
		background-color: #30bf78;
	}

	.item-check-yes-all {
		margin-bottom: 5px;
		border: 2px solid #30bf78;
		border-left: 0;
		border-top: 0;
		height: 12px;
		width: 6px;
		transform-origin: center;
		/* #ifndef APP-NVUE */
		transition: all 0.3s;
		/* #endif */
		transform: rotate(45deg);
	}

	.item-check .radio {
		border-top-left-radius: 50%;
		border-top-right-radius: 50%;
		border-bottom-right-radius: 50%;
		border-bottom-left-radius: 50%;
	}

	.item-check .radio .item-check-yes-b {
		border-top-left-radius: 50%;
		border-top-right-radius: 50%;
		border-bottom-right-radius: 50%;
		border-bottom-left-radius: 50%;
	}

	.hover-c {
		opacity: 0.6;
	}

	.itemBorder {
		border-bottom: 1px solid #e5e5e5;
	}
</style>
~~~

## 使用模版

~~~vue
                    <f-tree-picker-v2
                      ref="treePicker"
                      :multiple="false"
                      @select-change="areaSheetCallback"
                      title=""
                      @cancel="sheetArea"
                      @submit="submit"
                      :localdata="newList"
                      :showDialog="show"
                      valueKey="gridId"
                      textKey="title"
                      childrenKey="children"
                      v-if="show"
                    >
                  </f-tree-picker-v2>

//选择数据的时候触发
function areaSheetCallback(value: any) {
     form.orgId = value[0].name;
     form.townName = value[0].name;
     form.villageName = value[0].name;

      if( value[0].parentId == -1){
        form.townName = value[0].name;
        form.villageName ="";
      }else{
        form.townName = value[0].parentAreaName;
      }


      // 处理选择变化的回调
      console.log('Selected value:', value);
      console.log('Selected:', value[0].name);

    }

//关闭窗口的时候触发
function sheetArea() {
      // 处理取消选择的回调
      show.value = false;
    }


//点击确定提交的时候出发
function submit(){
    show.value = !show.value
}
~~~

## 使用数据

要注意，children不能为[]，只能为null

~~~js
let list =  [
          {
            title: "测试标题1",
            text: "测试标题1",
            key: 4000,
            value: 4000,
            parentAreaId: null,
            parentAreaName: null,
            gridId: null,
            isGridAdmin: 0,
            gridType: 0,
            fGridCode: null,
            children: [
              {
                title: "测试标题2",
                text: "测试标题2",
                key: 4501,
                value: 4501,
                parentAreaId: 230524504000,
                parentAreaName: "测试标题1",
                gridId: "0523",
                isGridAdmin: 0,
                gridType: 0,
                fGridCode: "4501",
                children: [
                  {
                    title: "测试标题3",
                    text: "测试标题3",
                    key: 4502,
                    value: 4502,
                    areaLevel: 3,
                    parentAreaId: 230524504501,
                    parentAreaName: "测试标题2",
                    gridId: "0524",
                    isGridAdmin: 0,
                    gridType: 0,
                    fGridCode: "4502",
                    children: null
                  },
                  {
                    title: "测试标题4",
                    text: "测试标题4",
                    key: 4503,
                    value: 4503,
                    areaLevel: 3,
                    parentAreaId: 4501,
                    parentAreaName: "测试标题2",
                    gridId: "0525",
                    isGridAdmin: 0,
                    gridType: 0,
                    fGridCode: "4503",
                    children: null
                  }
                ]
              },
              {
                title: "测试标题4",
                text: "测试标题4",
                key: 4504,
                value: 4504,
                areaLevel: 2,
                parentAreaId: 4000,
                parentAreaName: "测试标题1",
                gridId: "0526",
                isGridAdmin: 0,
                gridType: 0,
                fGridCode: "4504",
                children: null
              },
              {
                title: "测试标题5",
                text: "测试标题5",
                key: 4512,
                value: 4512,
                areaLevel: 5,
                parentAreaId: 4000,
                parentAreaName: "测试标题1",
                gridId: "0527",
                isGridAdmin: 1,
                gridType: 1,
                fGridCode: "4512",
                children: null
              }
            ]
          }
]
~~~

## 官方文档-树形层级选择器

### 简介

为统一样式而生，树形层级选择器，picker弹窗形式的，样式和比例参照uniapp的picker和uni-data-picker组件

* 支持单选、多选、父级选择，当然也支持单层选择
* 支持Object对象属性自定义映射
* 支持显示全部选中、部分选中、未选中三种状态
* 支持快速自定义简单样式（分割线、按钮、标题、对齐等），深入样式可复写css

### 使用方法

在 `script` 中引入组件

``` javascript
	import baTreePicker from "@/components/ba-tree-picker/ba-tree-picker.vue"
	export default {
		components: {
			baTreePicker
		}
```

在 `template` 中使用组件

``` javascript
	<ba-tree-picker ref="treePicker" :multiple='false' @select-change="selectChange" title="选择城市"
		:localdata="listData" valueKey="value" textKey="label" childrenKey="children" />
```

在 `script` 中定义打开方法，和选择监听

``` javascript
		methods: {
			// 显示选择器
			showPicker() {
				this.$refs.treePicker._show();
			},
			//监听选择（ids为数组）
			selectChange(ids, names) {
				console.log(ids, names)
			}
		}
```

在 `template` 中调用打开

``` javascript
	<view @click="showPicker">调用选择器</view>
```

### 属性

| 属性名       |  类型   |  默认值  |                                               说明 |
| :----------- | :-----: | :------: | -------------------------------------------------: |
| localdata    |  Array  |    []    | 源数据，目前支持tree结构，后续会考虑支持扁平化结构 |
| valueKey     | String  |    id    |              指定 Object 中 key 的值作为节点数据id |
| textKey      | String  |   name   |            指定 Object 中 key 的值作为节点显示内容 |
| childrenKey  | String  | children |                指定 Object 中 key 的值作为节点子集 |
| multiple     | Boolean |  false   |                                 是否多选，默认单选 |
| selectParent | Boolean |   true   |                           是否可以选父级，默认可以 |
| title        | String  |          |                                               标题 |
| titleColor   | String  |          |                                           标题颜色 |
| confirmColor | String  | #0055ff  |                                       确定按钮颜色 |
| cancelColor  | String  | #757575  |                                       取消按钮颜色 |
| switchColor  | String  |   #666   |                                   节点切换图标颜色 |
| border       | Boolean |  false   |                               是否有分割线，默认无 |



###  数据格式

注意：必须有id、name(id可通过valueKey来配置为其它键值，如value)字段，且唯一

``` json
[
    {
        id: 1,
        name: '公司1',
        children: [{
            id: 11,
            name: '研发部',
            children: [{
                id: 111,
                name: '张三',
                
            },{
                id: 112,
                name: '李四',
                
            }]
        },{
            id: 12,
            name: '综合部',
            
        } ]
    },
    {
        id: 2,
        name: '公司2',
        children: [{
            id: 21,
            name: '研发部',
            
        },{
            id: 22,
            name: '综合部',
            
        },{
            id: 23,
            name: '财务部',
            
        }, ]
    },
    {
        id: 3,
        name: '公司3'
    },
    {
        id: 4,
        name: '公司4',
        children: [{
            id: 41,
            name: '研发部',
            
        }]
    }
]
```

</details>

### 方法

| 方法名  | 参数 | 默认值 |       说明 |
| :------ | :--: | :----: | ---------: |
| _show() |      |        | 显示选择器 |
| _hide() |      |        | 隐藏选择器 |