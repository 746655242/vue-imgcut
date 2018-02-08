# vue-imgCut

vue mobile image clipping



## Install

``` bash
# install dependencies
npm install vue-imgCut --save
```

## Usage


``` javascript
import {imgCut} from 'vue-imgCut'

```

### Using the v-touch directive

``` html

<imgCut ref="Uppicinput" @callback="callback" :width="200" :height="200"></imgCut>


export default {
    name: 'picUp',
    components:{
       imgCut
	},
	methods:{
		callback(img){
			console.log(img,)
		}
	}
	
}

```


