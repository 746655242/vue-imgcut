# vue-imgcut

vue mobile image clipping

## Install

``` bash
# install dependencies
npm install vue-imgcut --save
```

###  vue-imgcut Instructions for use 

``` html
<imgCut ref="Uppicinput" @callback="callback" :width="200" :height="200"></imgCut>

import {imgCut} from 'vue-imgcut'
export default {
	components:{
		imgCut
	},
	methods:{
		callback(img){
			console.log(imgss)
		}
	}
}

```


