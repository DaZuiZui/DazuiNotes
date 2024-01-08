# Vue调用子组件内容和子组件方法失效

~~~java
      this.$nextTick(()  => {
        this.$refs.dictItemModal.showModalFlag = true;
      });
~~~

this.$nextTick 是为了防止我们在DOM还没有完成更新之前进行操作，使用这个就是确保了我们的DOM完成了更新在进行操作，在我们更新Vue组件的数据时候，我们的Vue会异步的执行DOM的更新，使用nextTIck就是等待这个更新后在操作，防止我们的组件还没完成渲染就对我们的$ref进行操作。