# Elemeui 走马灯，鼠标悬停不轮播了

## 问题分析

因为Element ui有鼠标悬停事件，只需要重写悬停事件即可。

## 方案解决

给轮播图设置ref name。	

~~~java
 <el-carousel ref="carousel" :autoplay="true" height="100vh" :interval="carouselItems[currentIndex].duration" arrow="always" @change="handleCarouselChange" 
      trigger="click" @mouseenter.native="delHandleMouseEnter"
      >
        <el-carousel-item v-for="(item, index) in carouselItems" :key="index" label=" ">
          <div class="carousel-item-content" :style="{ backgroundImage: 'url(' + item.imageUrl + ')' }">
            <p class="carousel-text">Hi This is GuangShaOJ By TLM Team</p> 
          </div>
        </el-carousel-item>
      </el-carousel>
~~~

清空鼠标悬停事件

~~~java
    methods: {
      handleCarouselChange(index) {
        this.currentIndex = index;
      },
      delHandleMouseEnter() {
        this.$refs.carousel.handleMouseEnter = () => {};
      },
    },
    mounted() {
      this.$refs.carousel.handleMouseEnter = () => {};
    },
~~~

