<template>
 <div id="content" 
    v-bind:class="{'content':configs.bgAllBool}"
    v-bind:style="{
      width:configs.bgAllBool ? configs.window_w +'px' : 'auto',
      height:configs.bgAllBool ? configs.window_h +'px' : 'auto',
      left:configs.bgAllBool && 0
    }">
    <div id="canvas"
      v-bind:style="{'transform':'scale('+configs.scale+')  translate('+ -(1-configs.scale) * 950+'px,'+ -(1-configs.scale) * 400+'px)'}"
      v-bind:class="{grid:configs.bg === 'grid'}" >    
      <!--遍历组件数组:图片-img,横线-lien_row,竖线-line_colu,柱子-pillar -->
      <div class="comp-item"
        v-for="(item, index) in edrawComps" :key="index"
        @click="selectItem($event, index)"
        @dragstart="!configs.bgAllBool && item.style.isFixed === 'false' && dragComp($event, item, 'start', index)"
        @drag="!configs.bgAllBool && item.style.isFixed === 'false' && dragComp($event, item, 'drag', index)"
        @dragend="!configs.bgAllBool && item.style.isFixed === 'false' && dragComp($event, item,'end', index)"
        v-bind:style="{
            width:item.style.width +'px',
            height:item.style.height +'px',
            top:(item.style.top - rect.y) +'px',
            left:(item.style.left - rect.x) +'px',
            position:item.style.position,
            borderWidth:item.style.borderWidth + 'px',
            transform: 'rotate('+ item.style.rotate +'deg)'
        }"
        v-bind:class="{
          active:item.isActive,
          roomActive:item.type === 'room'}"
        v-bind:draggable="!configs.bgAllBool && item.isActive">

        <!-- 1.img -->
        <img class="comp-element comp-img icon"
          v-if="item.type === 'img'" 
          v-bind:src="item.icon" 
          v-bind:style="{
              width:item.style.width +'px',
              height:item.style.height +'px',
              borderRadius:item.style.borderRadius +'%'
          }" 
          v-bind:title="item.name"
          v-bind:alt="item.name" 
          v-bind:class="{
            active:item.isActive, 
            isShadow:item.style.isApplyShadow ==='true',
            multipleActive:item.multipleActiveBool}">

        <!-- 2.line_colu  -->
        <div class="comp-element line comp-line_colu"
          v-if="item.type === 'line_colu'"
          v-bind:style="{
            height:item.style.height +'px'
          }"
          v-bind:class="{
            active:item.isActive, 
            isShadow:item.style.isApplyShadow ==='true',
            multipleActive:item.multipleActiveBool}">
        </div>

        <!-- 3.line_row  -->
        <div class="comp-element line comp-line_row"
          v-if="item.type === 'line_row'"
          v-bind:style="{
            with:item.style.width +'px'
          }"
          v-bind:class="{
            active:item.isActive, 
            isShadow:item.style.isApplyShadow ==='true',
            multipleActive:item.multipleActiveBool}">
        </div>

        <!-- 4.柱子 -->
        <div class="comp-element line comp-pillar comp-pillar-shadow "
          v-if="item.type === 'pillar'"
          v-bind:style="{
            height:item.style.height +'px',
            with:item.style.width +'px'
          }"
          v-bind:class="{
            active:item.isActive, 
            isShadow:item.style.isApplyShadow ==='true',
            multipleActive:item.multipleActiveBool}">

        </div>

        <!-- 5.盒子 -->
        <div class="comp-element line comp-room"
          v-if="item.type === 'room'"
          v-bind:style="{
            width:item.style.width +'px',
            height:item.style.height +'px',
          }"
          v-bind:class="{
            active:item.isActive, 
            isShadow:item.style.isApplyShadow ==='true',
            multipleActive:item.multipleActiveBool}">
          <span class="comp-room-inset"
            v-bind:style="{
              width:item.style.width +'px',
              height:item.style.height +'px',
            }"></span>
        </div>
        
        <!-- 6.设备 -->
        <div class="comp-element device comp-device"
          v-if="item.type === 'device'"
          v-bind:style="{
            width:item.style.width +'px',
            height:item.style.height +'px',
          }"
          v-bind:class="{
            active:item.isActive, 
            isShadow:item.style.isApplyShadow ==='true',
            multipleActive:item.multipleActiveBool}">
        </div>

        <!-- 辅助 -->
        <span 
          class="assist"
          v-if="!configs.bgAllBool && item.isActive && ['pillar'].indexOf(item.type) === -1">

          <span class="adR" 
            v-if="['line_colu'].indexOf(item.type) === -1"
            draggable="true"
            v-bind:style="{
              left:(item.style.width - 6 + item.style.borderWidth * 2) +'px',
              top:(item.style.height /2 - 6)+'px',
            }"
            @dragstart.stop="resizeByDragComp($event, item, 'start','r', index)"
            @drag.stop="resizeByDragComp($event, item, 'drag','r', index)"
            @dragend.stop="resizeByDragComp($event, item,'end', 'r', index)"></span>
          
          <span class="adL"
            v-if="['line_colu'].indexOf(item.type) === -1"
            draggable="true"
            v-bind:style="{
              left:(-6) +'px',
              top:(item.style.height /2 - 6)+'px',
            }"
            @dragstart.stop="resizeByDragComp($event, item, 'start','l', index)"
            @drag.stop="resizeByDragComp($event, item, 'drag','l', index)"
            @dragend.stop="resizeByDragComp($event, item,'end', 'l', index)"></span>
          
          <span class="adT"
            v-if="['line_row'].indexOf(item.type) === -1"
            draggable="true"
            v-bind:style="{
              left:(item.style.width /2 -6) +'px',
              top:(-6)+'px',
            }"
            @dragstart.stop="resizeByDragComp($event, item, 'start','t', index)"
            @drag.stop="resizeByDragComp($event, item, 'drag','t', index)"
            @dragend.stop="resizeByDragComp($event, item,'end', 't', index)"></span>

          <span class="adB"
            v-if="['line_row'].indexOf(item.type) === -1"
            draggable="true"
            v-bind:style="{
              left:(item.style.width /2 -6) +'px',
              top:(item.style.height - 6 + item.style.borderWidth * 2) +'px',
            }"
            @dragstart.stop="resizeByDragComp($event, item, 'start','b', index)"
            @drag.stop="resizeByDragComp($event, item, 'drag','b', index)"
            @dragend.stop="resizeByDragComp($event, item,'end', 'b', index)"></span>

          <span class="trans" 
            draggable="true"
            v-bind:style="{
              left:(item.style.width + 25) +'px',
              top:(item.style.height / 2 - 11)+'px',
            }"
            @dragstart.stop="trans($event, item, 'start', index)"
            @drag.stop="trans($event, item, 'drag', index)"
            @dragend.stop="trans($event, item,'end', index)">
            <img src="./../assets/icon/translate.svg" alt="">
          </span>
        </span>
      </div>
  </div>
  <div class="slide" v-if="!configs.bgAllBool">
    <label for="">缩放</label>
    <input 
    type="range" 
    class="slider" 
    v-model="configs.scale" 
    id="vol" 
    step="0.01" 
    name="vol" 
    min="0.7" 
    max="1.0">
  </div>
  <div class="screen" v-if="configs.bgAllBool">
    <img @click="screen" src="./../assets/icon/screen_cancel.svg" alt="右对齐" title="右对齐">
  </div>

 </div>
</template>
       
<script>

export default {
  name: 'EditorCanvas',
  props: {
    currentActiveIndex:Number, // 编辑状态管理
    edrawComps: Array, //绘制组件（图片）列表
    configs:Object
  },
  data() {
    return {
      name:"age",
      timerState:{
        lastTime:0,
        waitTime:10
      },
      rect:{
        x:0,
        y:0
      }
    }
  },
  mounted:function() {
    let dom = document.querySelector('#content');
    this.rect = dom && dom.getBoundingClientRect();
  },
  methods: {
    selectItem:function(event, index) {
      this.$emit('initCompsState',index, event);
    },
    // 拖拽图片，注释了节流优化
    dragComp:function(event, comp, state, index) {
      if(state === 'start') {
        this.$emit('initCompsState',index);
        this.$emit('dragComp',event, comp, state, index, this.rect)
        comp.isActive = true;
      } else {
        this.$emit('dragComp',event, comp, state, index, this.rect)
      }
    },
    resizeByDragComp:function(event, comp, arrow, state, index) {
      if(event && !event.clientX && !event.clientY) return;
      if(state === 'start') {
        this.$emit('resizeByDragComp',event, comp, arrow, state, index)
      } else {
        this.$emit('resizeByDragComp',event, comp, arrow, state, index)
      }
    },
    trans:function(event, comp, state, index) {
      if(event && !event.clientX && !event.clientY) return;
      this.$emit('trans',event, comp, state, index)
    },
    screen:function() {
      this.$emit('screen')
    }
  }
}


</script>

<style scoped>
#content.content {
  position: fixed;
  left: 10px;
  top: 0px;
  z-index: 9999;
  text-align: center;
}
#content, #canvas {
  position: relative;
  width: 100%;
  height: 822px;
  background: whitesmoke;
  overflow: auto;
}
.slide {
  position: fixed;
  top: 17px;
  left: 313px;
  z-index: 10000;
  user-select: none;
  font-size: 14px;
}
.screen {
  position: absolute;
  right: 30px;
  top:10px;
  cursor: pointer;
  opacity: 0.1;
}
.screen:hover {
  opacity: 0.9;
}
.screen img {
  width: 30px;
  height: 30px;
}
#vol {
  margin-left: 10px; 
}

.slider {
  -webkit-appearance: none;
  width: 150px;
  height: 6px;
  color:#2c3e50;
  background: #d3d3d3;
  outline: none;
  opacity: 0.7;
  -webkit-transition: .2s;
  transition: opacity .2s;
}

.slider:hover {
  opacity: 1;
}

.slider::-webkit-slider-thumb {
  -webkit-appearance: none;
  appearance: none;
  width: 25px;
  height: 25px;
  border-radius: 50%;
  background: red;
  cursor: pointer;
}

.slider::-moz-range-thumb {
  width: 25px;
  height: 25px;
  background: red;
  cursor: pointer;
}

img {
  border:1px solid transparent;
}
.comp-item {
  user-select: none;
  cursor: move;
  z-index: 1;
}
.active {
  z-index: 11;
}
.roomActive {
  z-index: 0;
}
.multipleActive {
  border:1px solid red ;
}
.comp-item .active {
  border:1px solid rgba(225,0,00,0.8);
}
.comp-item .comp-room.active {
  border:4px solid white;
  box-shadow: 1px 0px 8px 0px rgba(255,0,0,0.6);
}
.comp-device {
  background: yellowgreen;
}

#canvas {
    width: 1900px;
    height: 900px;
    background: #eeeeee;
      transform: translate(100px, 200px);
}
#canvas.grid {
    background-image: linear-gradient(rgba(200,205,208,.3) 1px,transparent 0),
                      linear-gradient(90deg,rgba(200,205,208,.3),1px,transparent 0),
                      linear-gradient(#c8cdd0 1px,transparent 0),
                      linear-gradient(90deg,#c8cdd0 1px,transparent 0);
    background-size: 20px 20px,20px 20px,100px 100px,100px 100px;
}
.control {
  position: fixed;
  bottom: 100px;
  right: 30px;
}
.control .del {
  cursor: pointer;
  width: 30px;
  height: 30px;
  opacity: 0.7;
  padding: 5px;
  border-radius: 50%;
  background: #dddddd;
}
.control .del:hover {
  opacity: 1;
  box-shadow: 0px 0px 10px red;
}
.delDef {
  opacity: 0.3;
}
.comp-img.isShadow {
  box-shadow: -1px 1px 10px 5px rgba(0,0,0,0.3);
}
.comp-line_colu {
  width: 5px;
  background: #ffffff;

}
.comp-line_colu.isShadow {
  box-shadow: 0px 0px 14px -1px rgba(0,0,0,1);
}
.comp-line_row {
  height: 5px;
  background: #ffffff;
}
.comp-line_row.isShadow {
  box-shadow: 0px 0px 14px -1px rgba(0,0,0,1);
}
.comp-pillar {
  width: 15px;
  height: 15px;
  background: #fff;
}
.comp-pillar-shadow.isShadow {
  box-shadow: 0px 0px 3px 1px rgba(0,0,0,0.4);
}
.comp-room {
  border: 4px solid #ffffff;
}
.comp-room.isShadow {
  box-shadow: 1px 0px 8px 0px rgba(0,0,0,0.6);
}
.comp-room-inset {
  display: inline-block;
}
.comp-room-inset.isShadow {
  box-shadow: inset 0px 0px 26px -11px rgba(13,13,13,0.8);
}

.assist span {
  position: absolute;
  width: 10px;
  height: 10px;
  border:1px solid red;
  display: inline;
  background: #fff;
}
.assist .trans {
  width: 22px;
  height: 22px;
  border:none;
  background: transparent;
  border-radius: 50%;
  cursor: pointer;
}
.assist .trans img {
  width: 22px;
  height: 22px;
}
.adR, .adL {
  cursor: ew-resize;
}
.adT, .adB {
  cursor: ns-resize;
}
</style>
