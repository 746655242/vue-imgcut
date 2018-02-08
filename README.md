# vue-imgcut

vue mobile image clipping

## Install

``` bash

npm install vue-imgcut --save
```

###  vue-imgcut Instructions for use 

``` html
<imgCut ref="Uppicinput" @callback="callback" :width="200" :height="200">
	<div>上传按钮</div>
</imgCut>

<img :src="imgsrc">



import {imgCut} from 'vue-imgcut'
export default {
	data(){
		return{
			imgsrc:'',
		}
	},
	components:{
		imgCut
	},
	methods:{
		callback(img){
			thie.imgsrc=img
			//console.log(imgss)
		}
	}
}

```


