# vue-imgcut

vue mobile image clipping



## Install

``` bash
# install dependencies
npm install vue-imgcut --save
```

## Usage


``` javascript
import {imgCut} from 'vue-imgcut'

```

### Using the v-touch directive

``` html

<imgCut ref="Uppicinput" @callback="callback" :width="200" :height="200"></imgCut>


export default {
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


