# 使用antdv表示树形结构

```java
      <a-form-item style="float: left;"  >
        <a-tree-select style="width:200px" v-model="val" showSearch allow-clear>
          <a-tree-node v-for="item in treeData" :value="item.key"   :title="item.title">
            <a-tree-node v-for="tmp in item.children" :value="tmp.key"  :title="tmp.title">、
              <a-tree-node v-for="tmp1 in tmp.children" :value="tmp1.key"   :title="tmp1.title">
              </a-tree-node>
            </a-tree-node>
          </a-tree-node>
        </a-tree-select>
      </a-form-item>
```

如果还有子节点，那么久继续写a-tree-node

~~~java
        treeData: [
        {
          title: "红卫农场",
          text: "红卫农场",
          key: 230524504000,
          value: 230524504000,
          areaLevel: 1,
          parentAreaId: null,
          parentAreaName: null,
          gridId: null,
          isGridAdmin: 0,
          gridType: 0,
          fGridCode: null,
          children: [
            {
              title: "红卫农场第一管理区",
              text: "红卫农场第一管理区",
              key: 230524504501,
              value: 230524504501,
              areaLevel: 2,
              parentAreaId: 230524504000,
              parentAreaName: "红卫农场",
              gridId: "1621328826774720523",
              isGridAdmin: 0,
              gridType: 0,
              fGridCode: "230524504501",
              children: [
                {
                  title: "红卫农场第一管理区第一分区",
                  text: "红卫农场第一管理区第一分区",
                  key: 230524504502,
                  value: 230524504502,
                  areaLevel: 3,
                  parentAreaId: 230524504501,
                  parentAreaName: "红卫农场第一管理区",
                  gridId: "1621328826774720524",
                  isGridAdmin: 0,
                  gridType: 0,
                  fGridCode: "230524504502",
                  children: []
                },
                {
                  title: "红卫农场第一管理区第二分区",
                  text: "红卫农场第一管理区第二分区",
                  key: 230524504503,
                  value: 230524504503,
                  areaLevel: 3,
                  parentAreaId: 230524504501,
                  parentAreaName: "红卫农场第一管理区",
                  gridId: "1621328826774720525",
                  isGridAdmin: 0,
                  gridType: 0,
                  fGridCode: "230524504503",
                  children: []
                }
              ]
            },
            {
              title: "红卫农场第二管理区",
              text: "红卫农场第二管理区",
              key: 230524504504,
              value: 230524504504,
              areaLevel: 2,
              parentAreaId: 230524504000,
              parentAreaName: "红卫农场",
              gridId: "1621328826774720526",
              isGridAdmin: 0,
              gridType: 0,
              fGridCode: "230524504504",
              children: []
            },
            {
              title: "红卫农场第三管理区",
              text: "红卫农场第三管理区",
              key: 230524504512,
              value: 230524504512,
              areaLevel: 5,
              parentAreaId: 230524504000,
              parentAreaName: "红卫农场",
              gridId: "1621328826774720527",
              isGridAdmin: 1,
              gridType: 1,
              fGridCode: "230524504512",
              children: []
            }
          ]
        }
      ],
~~~

