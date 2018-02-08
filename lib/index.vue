<template>
    <div id="uppic">
        <slot></slot>
        <input class="file" type="file" @change="getsrc" ref="file">
        <div id="uppicBox" ref="uppicBox"  v-show="isuppicBox">
            <div class="imageBox">
                <img id="pic-img" ref="picImg" :src="img.src" :style="{'transform':'translate3d('+transform.x+'px,'+transform.y+'px,0px) scale('+transform.scale+','+transform.scale+')','transition':'transform '+duration+'ms','width':img.w+'px','height':img.h+'px','margin-left':img.w/-2+'px','margin-top':img.h/-2+'px'}">
                <div class="thumbBox box" 
                    :class="{'bgshadow':isbg}" 
                    ref="thumbBox" 
                    :style="{'width':Cwidth+'px','height':Cheight+'px','margin-left':0-Cwidth/2+'px','margin-top':0-Cheight/2+'px'}">
                    
                    <div class="angle1"></div>
                    <div class="angle2"></div>
                    <div class="angle3"></div>
                    <div class="angle4"></div>
                    <div class="line1"></div>
                    <div class="line2"></div>
                    <div class="line3"></div>
                    <div class="line4"></div>
                </div>
            </div>

            <div class="bottom" @click.stop>
                <span  @click="cancel">取消</span>
                <span  @click="clip" class="fr">选取</span>
            </div>
        </div>
    </div>
</template>
<script>

import "./exif.js"
import Hammer from "hammerjs"
export default {
    name:'Uppic',
    data(){
        return{  
            transform:{
                'scale':1,
                'x':0,
                'y':0
            },
            isuppicBox:false,
            isbg:false,
            duration:0,
            Cwidth:0,
            Cheight:0,
            img:{
                Yh:0,
                Yw:0,
                h:0,
                w:0,
                src:'',
                Orientation:1
            },
            x:0,
            y:0,
            initScale:0,

        }
    },
    props:{
        callback:Function,
        width:{
            type:Number,
            default:200,
            validator:function(value){
                return value!=0
            }
        },
        height:{
            type:Number,
            default:200
        },
        data:{
            type:Object,
            default:function(){
                return {}
            }

        }
    },
    methods:{ 
        fileclick(){
            this.$refs.file.click()
        },
        getsrc(e){
            let me=this
            let files=e.target.files[0]
            let fileReader = new FileReader()
			fileReader.onloadend=function(e){
                me.img.src=e.target.result
                me.$refs.picImg.onload=me.init;
            }
			fileReader.readAsDataURL(files)
        },
        init(e){
          
            let me=this
            me.isuppicBox=true
            let picImg=me.$refs.picImg
            me.img.Yw=picImg.naturalWidth
            me.img.Yh=picImg.naturalHeight

			EXIF.getData(picImg, function(){
                me.img.Orientation=EXIF.getTag(this, 'Orientation')||1;
                me.sizeimg()
            });	
           
            document.body.onresize=me.sizeimg;
            
            let mc=new Hammer.Manager(this.$refs.uppicBox)
            mc.add(new Hammer.Pan({ threshold: 0, pointers: 0 }));
			mc.add(new Hammer.Pinch({threshold: 0})).recognizeWith(mc.get('pan'));
            mc.add(new Hammer.Tap({ event: 'doubletap', taps: 2 }));
            mc.add(new Hammer.Tap());

            mc.on("doubletap",me.doubletap);
			mc.on("pinchstart pinchmove",me.pinchin);
			mc.on("panstart panmove",me.Pan);
			mc.on('panend',me.panend);
            
        },
        sizeimg:function(){
            let me=this
            let w=document.body.clientWidth
            let h=document.body.clientHeight
            let Yh=me.img.Yh
            let Yw=me.img.Yw
            let Width,Height
            
            /* 设置截取框长宽 */
            if(w*me.height<h*me.width){ 
                Width=w-40;
                Height= Width/me.width*me.height 
            }else{ 
                Height=h-40;
                Width= Height/me.height*me.width 
            }
       
            this.Cwidth=Width;
            this.Cheight=Height;				

           
            /*设置原图片 iPhone图片 */
            if(Yw*Height<Yh*Width){
                /* 以宽为基准缩放 */
                me.img.w=Width
                me.img.h=this.browser()=='iphone'?Yw*Width/Yh:Yh*Width/Yw
            
            }else{
                /* 以高为基准缩放 */
                me.img.h=Height
                me.img.w=this.browser()=='iphone'?Yh*Height/Yw :Yw*Height/Yh
                
            }
            
           
        },
        touchTU:function(evt){
			// var touch = evt.originalEvent.touches[0]; //获取第一个触点
			// this.x = Number(touch.pageX);
			// this.y=Number(touch.pageY);			
		},
        doubletap(e){
            let scale=this.transform['scale']
            if(scale<4){
                this.transform['scale']=scale*1.4;
            }
            console.log('双击')
        },
        pinchin(ev){
            if(ev.type == 'pinchstart') {
				this.initScale = this.transform.scale || 1;
            }
            let scale=this.initScale*ev.scale
            if(scale<4){
                this.transform.scale=scale
            }
            console.log('缩放')
        },
        Pan(ev){
            if(ev.type == 'panstart') {
				this.x=this.transform['x']||0;
				this.y=this.transform['y']||0;
            }
			this.transform.x=this.x+ev.deltaX;
            this.transform.y=this.y+ev.deltaY;
            
            this.isbg=true
            console.log('移动')
        },
        panend(){
            let me=this
            let scale=this.transform.scale
			let tx=this.transform.x
			let ty=this.transform.y

            let w=this.img.w*scale;
			let h=this.img.h*scale;
            let ww=this.Cwidth;
            let hh=this.Cheight;
            
            let left=(w-ww)/2;
			let right=(ww-w)/2;
			let top=(h-hh)/2;
			let bottom=(hh-h)/2;

            if(scale<1){
                scale=1
                if(h>w){tx=0}else{ty=0}
            }else{
                if(tx>left){
					tx=left;
					console.log('过left底线');
				}else if(tx<right){
					tx=right;
					console.log('过right底线');
				}
				if(ty>top){
					ty=top;
					console.log('过top底线');
				}else if(ty<bottom){
					ty=bottom;
					console.log('过bottom底线');
				}    
            }

            this.duration=300;
            this.transform.scale=scale
			this.transform.x=tx
			this.transform.y=ty
            
            setTimeout(function(){
                me.isbg=false
                me.duration=0;
            },300)           
            
        },
        /* 取消 */
        cancel(){
            this.isuppicBox=false
            this.img.src=''
            this.transform={
                'scale':1,
                'x':0,
                'y':0
            }
            this.$refs.file.value=''
        },
        /*裁剪 */
        clip(){
            let me=this
            let image=me.$refs.picImg;
            let canvas = document.createElement("canvas");
            let context = canvas.getContext("2d");  
            //缩放值 
			let scale=this.transform.scale;
			let tx=this.transform.x;
            let ty=this.transform.y;
            
            
            
            let top=(this.img.h*scale-this.Cheight)/2
            let left=(this.img.w*scale-this.Cwidth)/2
            let bili
            if(this.browser()=='iphone'){
                bili=this.img.Yh/(this.img.w*scale)
            }else{
                bili=this.img.Yh/(this.img.h*scale)
            }

            let sw=this.Cwidth*bili
            let sh=this.Cheight*bili
            let sx=(left-tx)*bili
            let sy=(top-ty)*bili
            let dw=this.width
            let dh=this.height

            canvas.width = dw;//设置canvas宽
            canvas.height = dh;//设置canvas宽
            
            let Orientation=me.img.Orientation
            if(Orientation>1){
                this.drawPhoto(image,Orientation,function(img,shuoxiao){
                    context.drawImage(img,sx/shuoxiao,sy/shuoxiao,sw/shuoxiao,sh/shuoxiao,0,0,dw,dh);//向画布上绘制图像
                    var imageData = canvas.toDataURL('image/jpg');
                    me.$emit('callback',imageData,me.data)
                    me.cancel()
                })
            }else{
                context.drawImage(image,sx,sy,sw,sh,0,0,dw,dh);//向画布上绘制图像
                var imageData = canvas.toDataURL('image/jpg');//设置格式  
                this.$emit('callback',imageData,me.data)
                this.cancel()
            }
        },
        drawPhoto:function(img,dir,next){
			 var image=new Image();
			 image.onload=function(){
				  var degree=0,drawWidth,drawHeight,width,height,shuoxiao;
				  drawWidth=this.naturalWidth;
				  drawHeight=this.naturalHeight;
				  //以下改变一下图片大小
				  var maxSide = Math.max(drawWidth, drawHeight);
				  if (maxSide > 1024) {
					var minSide = Math.min(drawWidth, drawHeight);
					shuoxiao=maxSide/1024;
					minSide = minSide / maxSide * 1024;
					maxSide = 1024;
					if (drawWidth > drawHeight) {
					  drawWidth = maxSide;
					  drawHeight = minSide;
					} else {
					  drawWidth = minSide;
					  drawHeight = maxSide;
					}
				  }
				
				var canvas = document.createElement("canvas");
				canvas.width=width=drawWidth;
			    canvas.height=height=drawHeight; 
			    var context=canvas.getContext('2d');
				switch(dir){
                    //iphone横屏拍摄，此时home键在左侧
                    case 3:
                    degree=180;
                    drawWidth=-width;
                    drawHeight=-height;
                    break;
                    //iphone竖屏拍摄，此时home键在下方(正常拿手机的方向)
                    case 6:
                    canvas.width=height;
                    canvas.height=width; 
                    degree=90;
                    drawWidth=width;
                    drawHeight=-height;
                    break;
                    //iphone竖屏拍摄，此时home键在上方
                    case 8:
                    canvas.width=height;
                    canvas.height=width; 
                    degree=270;
                    drawWidth=-width;
                    drawHeight=height;
                    break;
                }
			  
			//使用canvas旋转校正
			  context.rotate(degree*Math.PI/180);
			  context.drawImage(this,0,0,drawWidth,drawHeight);			
                var imageData = canvas.toDataURL('image/jpg');//设置格式  
                var img=new Image()
                img.src=imageData
                img.onload=function(){
                    next(this,shuoxiao);  
                }
			}	
			image.src=img.src;		
        },
        browser(){
            let ua = navigator.userAgent.toLowerCase()
            let vendor=navigator.vendor.toLowerCase()
            if (/iphone|ipad|ipod/.test(ua)&&/apple/.test(vendor)) {
                return 'iphone'		
            }else if (/android/.test(ua)) {
                return "android"	
            }else{
                return 'pc'
            }
        }
    },
    watch:{
        
    }
}
</script>
<style lang="less">
html,body{height: 100%;}
#uppic{
    position: relative;
    width:100%;
    height:100%;
    min-height:20px;
    .file{
        position: absolute;
        width:100%;
        height: 100%;
        opacity:0;
        top:0;
        left:0;
        z-index:99;
        cursor: pointer;
    }
    #uppicBox{ 
        display: block;

        .thumbBox{ 
            width:100%; 
            border:1px solid #fff; 
            position:absolute; 
            top:50%; 
            left:50%; 
            box-shadow:0 0 0 1000px rgba(0, 0, 0, 0.8);
            .line1,.line2{
                width:100%;
                height: 1px;
                position: absolute;
                background: #fff;
                left:0;top:33.33%;
            }
            .line2{top:66.66%;}
            .line3,.line4{
                width:1px;
                height: 100%;
                position: absolute;
                background: #fff;
                left:33.33%;
                top:0;
            }
            .line4{left:66.66%;}

            .angle1,.angle2,.angle3,.angle4{ position:absolute;width: 15px;height: 15px;}
            .angle1{ left:-2px;top:-2px;border-top:2px solid #fff;border-left:2px solid #fff;}
            .angle2{ left:-2px;bottom:-2px;border-bottom:2px solid #fff;border-left:2px solid #fff;}
            .angle3{ right:-2px;top:-2px;border-top:2px solid #fff;border-right:2px solid #fff;}
            .angle4{ right:-2px;bottom:-2px;border-bottom:2px solid #fff;border-right:2px solid #fff;}

        }
        .bgshadow{
            box-shadow:0 0 0 1000px rgba(0, 0, 0, 0.2);
        }
        .bottom{ position:absolute; bottom:0; left:0; right:0; background:rgba(255,255,255,.05); font-size:16px; color:#fff; padding:10px 0; z-index:999999;
            span{ padding:10px; display:inline-block;float:left;}
            .fr{ float:right;}
        }
        #pic-img{ position:absolute; top:50%; left:50%; width: auto;}
    }
}
#uppicBox,.imageBox{ background:#000; position:fixed; top:0; left:0;right:0;bottom:0; z-index:999999; cursor:pointer;}

</style>

