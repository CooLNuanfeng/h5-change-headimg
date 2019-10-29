<template>
  <div class="warp-body">
      <div class="canvas-box" id="content">
          <canvas class="" id='cvs'></canvas>
      </div>
      <div class="head-list">
          <a href="javascript:;" class="slide-btn slide-prev" @click="prev"></a>
          <div class="head-items">
            <div id="head-hat">
                <div><img src='../assets/hat0.png' @click="hatClick(0)"/></div>
                <div><img src='../assets/hat1.png' @click="hatClick(1)"/></div>
                <div><img src='../assets/hat2.png' @click="hatClick(2)"/></div>
                <div><img src='../assets/hat3.png' @click="hatClick(3)"/></div>
                <div><img src='../assets/hat4.png' @click="hatClick(4)"/></div>
                <div><img src='../assets/hat5.png' @click="hatClick(5)"/></div>
                <div><img src='../assets/hat6.png' @click="hatClick(6)"/></div>
            </div>
          </div>
          <a href="javascript:;" class="slide-btn slide-next" @click="next"></a>
      </div>
      <div class="btns">
          <a href="javascript:;" class="btn-upload">
              <input class="o-btn1" id='upload' type='file' @change='doUpload()' style='opacity: 0;'>
          </a>
          <a href="javascript:;" class="btn-create" @click="createImg"></a>
      </div>
      <div v-show="createShow" class="create-show">
          <div class="create-img">
              <img id='export' alt='国庆专属头像' src='' />
              <div class="create-tips">
                  <i></i>长按保存头像
              </div>
              <div class="create-success">
                  <i></i>恭喜你头像定尺成功！
              </div>
          </div>
          <div class="create-btns">
              <a href="javascript:;" class="reset-btn" @click="reset"></a>
              <a href="javascript:;" class="share-btn" @click="share"></a>
          </div>
      </div>
      <div style="display:none">
        <img id='hat0' src='../assets/hat0.png' />
        <img id='hat1' src='../assets/hat1.png' />
        <img id='hat2' src='../assets/hat2.png' />
        <img id='hat3' src='../assets/hat3.png' />
        <img id='hat4' src='../assets/hat4.png' />
        <img id='hat5' src='../assets/hat5.png' />
        <img id='hat6' src='../assets/hat6.png' />
      </div>
      <div class="share-mask" v-show="shareMask" @click="changeMask">
          <img src="../assets/share_tips.png" alt="">
      </div>
  </div>
</template>

<script>
import {fabric} from 'fabric'
import EXIF from 'exif-js'
export default {
    name: 'headerFrame',
    data(){
        return {
            screenWidth: 0,
            curSlide: 0,
            slideLen: 0,
            slideStep: null,
            hatDiv: null,
            canvasFabric: null,
            hatFabric: null,
            imgFabric: null,
            imgFabricStartX: 0,
            imgFabricStartY: 0,
            createShow: false,
            shareMask: false
        }
    },
    mounted(){
        this.hatDiv = document.getElementById('head-hat')
        this.slideLen = this.hatDiv.children.length;
        this.slideStep = this.hatDiv.children[1].offsetLeft - this.hatDiv.offsetLeft;
        this.screenWidth = document.getElementById("content").scrollHeight
    },
    methods: {
        hatClick(id){
            let oImg = document.getElementById(`hat${id}`);
            if(!this.canvasFabric){
                this.canvasFabric = new fabric.Canvas("cvs", {
                    width: this.screenWidth,
                    height: this.screenWidth,
                });
                this.bindMoving()
            }
            if(this.hatFabric){
                this.canvasFabric.remove(this.hatFabric)
            }
            this.hatFabric = new fabric.Image(oImg, {
                top: 0,
                left: 0,
                scaleX: this.screenWidth / oImg.width,
                scaleY: this.screenWidth / oImg.width,
                selectable: false,
            });
            this.hatFabric.setControlVisible("bl", false);
            this.hatFabric.setControlVisible("tr", false);
            this.hatFabric.setControlVisible("tl", false);
            this.hatFabric.setControlVisible("mr", false);
            this.hatFabric.setControlVisible("mt", false);
            this.canvasFabric.add(this.hatFabric)
            this.canvasFabric.bringToFront(this.hatFabric)
            this.canvasFabric.renderAll()
        },
        next(){
            this.curSlide++
            if(this.curSlide > this.slideLen - 1){
                this.curSlide = this.slideLen - 1;
            }
            this.hatDiv.style.transform = `translateX(-${(this.slideStep*this.curSlide)}px)`
        },
        prev(){
            this.curSlide--
            if(this.curSlide < 0){
                this.curSlide = 0;
            }
            this.hatDiv.style.transform = `translateX(-${(this.slideStep*this.curSlide)}px)`
        },
        scaleImg(screenWidth,img){
            let scale = {
                scaleX: 0,
                scaleY: 0,
            }
            if(img.width > img.height){
                scale.scaleX = screenWidth / img.height,
                scale.scaleY = screenWidth / img.height
            }else{
                scale.scaleX = screenWidth / img.width,
                scale.scaleY = screenWidth / img.width
            }
            return scale
        },

        img2Cvs(img,angle){
            if(!this.canvasFabric){
                this.canvasFabric = new fabric.Canvas("cvs", {
                    width:this.screenWidth,
                    height: this.screenWidth,
                });
                this.bindMoving()
            }
            if(this.imgFabric){
                this.canvasFabric.remove(this.imgFabric)
            }
            let opts = this.scaleImg(this.screenWidth,img)
            this.imgFabric = new fabric.Image(img, {
                ...opts,
                selectable: false,
            });
            this.imgFabric.rotate(angle)
            
            this.canvasFabric.add(this.imgFabric)
            this.canvasFabric.bringToFront(this.hatFabric)
            this.canvasFabric.sendToBack(this.imgFabric)
            this.canvasFabric.renderAll()
            
        },
        bindMoving(){
            this.canvasFabric.on('mouse:down', (e) =>{
                if(!this.imgFabric) return
                this.imgFabricStartX = e.pointer.x - this.imgFabric.left
                this.imgFabricStartY = e.pointer.y - this.imgFabric.top
                this.canvasFabric.renderAll()
            })
            this.canvasFabric.on('mouse:move', (e) => {
                if(!this.imgFabric) return
                let curX = e.pointer.x;
                let curY = e.pointer.y;
                this.imgFabric.set({
                    left: curX - this.imgFabricStartX,
                    top: curY - this.imgFabricStartY
                })
                this.canvasFabric.renderAll()
            })
        },
        doUpload(){
            let _this = this
            let img = document.getElementById('export')
            let file = document.getElementById("upload").files[0];
            let angle,Orientation
            // console.log(file);
            if(!file) return
            var reader = new FileReader;
            if (file) {
                reader.readAsDataURL(file);
                reader.onload = function() {
                    img.src = reader.result;
                    img.onload = function() {
                        EXIF.getData(file, function () {
                            Orientation = EXIF.getTag(this, "Orientation");
                            switch(Orientation){  
                                case 6://需要顺时针（向左）90度旋转  
                                    _this.angle = 90
                                    break;  
                                case 1: 
                                    _this.angle = 0
                                    break;  
                                case 8://需要逆时针（向右）90度旋转  
                                    _this.angle = -90
                                    break; 
                                case 3: 
                                    _this.angle = 180
                                    break;
                            } 
                            _this.img2Cvs(img,angle)
                            document.getElementById("upload").value = null
                        });
                        
                    }
                }
            } else {
                img.src = ""
            }

            
        },
        createImg(){
            var exportImg = document.getElementById('export');
            this.canvasFabric.getContext().imageSmoothingEnabled = false
            this.canvasFabric.getContext().imageSmoothingQuality = 'hight'
            exportImg.setAttribute('src',this.canvasFabric.toDataURL({
                width: this.screenWidth,
                height: this.screenWidth,
                enableRetinaScaling: true
            }))
            this.createShow = true
        },
        reset(){
            this.canvasFabric.remove(this.imgFabric)
            this.canvasFabric.remove(this.hatFabric)
            this.createShow = false;
        },
        share(){
            this.shareMask = true
        },
        changeMask(){
            this.shareMask = false
        }
    }
}
</script>

<style lang="scss" socped>
@function vw($px){
    @return ($px / 750) * 100vw;
}
.warp-body{
    min-height: 100%;
    background: url(../assets/bg.png) center center no-repeat;
    background-size: cover;
    overflow: hidden;
}
.canvas-box{
    width: vw(320);
    height: vw(320);
    margin: vw(390) auto 0;
    background: #fff;
}
.btn-upload{
    display: flex;
    margin: 0 auto;
    margin-top: vw(53);
    width: vw(364);
    height: vw(102);
    border-radius: vw(51);
    background: url(../assets/upload_btn.png) no-repeat;
    background-size: cover;
}
.btns{
    padding-bottom: vw(100);
}
.btn-create{
    display: flex;
    margin: 0 auto;
    margin-top: vw(23);
    width: vw(364);
    height: vw(102);
    border-radius: vw(51);
    background: url(../assets/create_btn.png) no-repeat;
    background-size: cover;
}
.head-list{
    display: flex;
    height: vw(100);
    margin-top: vw(46);
    justify-content: center;
    align-items: center;
    .slide-btn{
        width: vw(46);
        height: vw(46);
    }
    .slide-prev{
        background: url(../assets/arrow_left.png) no-repeat;
        background-size: cover;
    }
    .slide-next{
        background: url(../assets/arrow_right.png) no-repeat;
        background-size: cover;
    }
    .head-items{
        margin: 0 vw(30);
        width: vw(380);
        height: vw(100);
        overflow: hidden;
        font-size: 0;
        text-align: left;
        white-space: nowrap;
        div{
            display: inline-block;
            width: vw(100);
            height: vw(100);
            background: #fff;
            margin-right: vw(40);
        }
        img{
            width: vw(100);
            height: vw(100);
            vertical-align: top;
            
        }
    }
}

.create-img{
    img{
        width: vw(320);
        height: vw(320);
        border-radius: vw(10);
    }
}

.create-show{
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    z-index: 30;
    display: flex;
    padding-top: vw(178);
    flex-direction: column;
    justify-items: center;
    align-items: center;
    background: url(../assets/success_bg.png) center center no-repeat;
    background-size: cover;
}
.create-tips{
    padding-top: vw(20);
    text-align: center;
    font-size: vw(30);
    color: #fff;
    i{
        display: inline-block;
        width: vw(30);
        height: vw(30);
        vertical-align: middle;
        margin-right: 5px;
        background: url(../assets/arrow_top.png) no-repeat;
        background-size: contain;
    }
}
.create-success{
    margin: 0 auto;
    margin-top: vw(30);
    display: flex;
    justify-content: center;
    align-items: center;
    width: vw(400);
    height: vw(140);
    background: #FFFADF;
    border: 1px solid #F8B327;
    font-size: vw(24);
    color: #333;
    i{
        display: inline-block;
        width: vw(44);
        height: vw(44);
        background: url(../assets/success.png) no-repeat;
        background-size: cover;
    }
}
.create-btns{
    display: flex;
    padding: vw(193) vw(80) 0;
    justify-content: space-between;
    align-items: center;
    a{
        display: inline-block;
        width: vw(264);
        height: vw(90);
    }
    .reset-btn{
        background: url(../assets/re_upload_btn.png) no-repeat;
        background-size: cover;
    }
    .share-btn{
        background: url(../assets/share_btn.png) no-repeat;
        background-size: cover;
    }
}
.share-mask{
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background: rgba(0,0,0,.7);
    z-index: 100;
    img{
        margin-top: vw(20);
        width: 100%
    }
}
</style>