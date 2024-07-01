# Element UI 分页更改背景颜色

~~~vue
          <el-pagination
            background
            layout="prev, pager, next"
            :total="1000"
            class="custom-pagination"
          >
          </el-pagination>

~~~

~~~css
<style>
.el-pagination.is-background .btn-next,
.el-pagination.is-background .btn-prev,
.el-pagination.is-background .el-pager li {
  background-color: #fff !important;
}

.el-pagination.is-background .el-pager li.active {
  color: #fff !important; /* 设置选中字体颜色为白色 */
  background-color: rgba(255, 46, 109, 1) !important; /* 设置选中背景色为粉色 */
}

</style>
~~~

