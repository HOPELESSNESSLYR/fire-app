<template>
  <view>

    <p class="error">{{ error }}</p>

    <p class="decode-result">
      扫描结果: <b>{{ result }}</b>
    </p>
		

    <view v-if="qrcode">
      <qrcode-stream
        :constraints="selectedConstraints"
        :track="trackFunctionSelected.value"
        :formats="selectedBarcodeFormats"
        @error="onError"
        @detect="onDetect"
        @camera-on="onCameraReady"
      />
    </view>
		
		<p class="resultinfo">
		  <b>{{ resultinfo }}</b>
		</p>
		
			<view>
					<ul>
			      <li v-for="item in inforList" :key="item.recordId" class="liclass">
			        {{ item.deviceName }} - {{ item.checkRecords.slice(0,10 )}} - {{ item.qualified }}
			      </li>
			    </ul>
			</view>

		<button @click="goCheck" class="imageButton">去点检</button>
		</view>
</template>

<script setup lang="ts">
import { ref, computed, onMounted } from 'vue'	

/*** detection handling ***/
import { QrcodeStream, QrcodeDropZone, QrcodeCapture } from 'vue-qrcode-reader'


/*** select device ***/
const fireid = ref('')
// const result = ref('B-5F-001')			//测试数据
const result = ref('')
let qrcode = ref(true)
const	selectedCategory = ref('');

// new
const inforList = ref([])
const inforNo = ref(0)
const resultinfo = ref('')

// onMounted(()=>{
// 	cata()
// })
// ----------------------
const MyComponent = {
  components: {
    QrcodeStream,
    QrcodeDropZone,
    QrcodeCapture
  }
}
function onDetect(detectedCodes) {
  console.log(detectedCodes)
	const regex = /^[\u4e00-\u9fa5\d]/;

	if(!regex.test(result.value)){
		result.value = JSON.stringify(detectedCodes.map((code) => code.rawValue)).replace(/[^\u4e00-\u9fa5a-zA-Z0-9-_]/g, '');
	}else{
		result.value = JSON.stringify(detectedCodes.map((code) => code.rawValue)).replace(/[^\u4e00-\u9fa5a-zA-Z0-9-_]/g, '');
	}

	// uni.showModal({
	// 	content:result.value
	// })
	
	// result.value = JSON.parse (detectedCodes.map((code) => code.rawValue))
	// console.log(detectedCodes.map((code) => code.rawValue)[0])
	
	if(result.value!=undefined&& result.value!=''){
		qrcode.value= false
	}
	
	cata()
}

/*** select camera ***/
const selectedConstraints = ref({ facingMode: 'environment' })
const defaultConstraintOptions = [
  { label: 'rear camera', constraints: { facingMode: 'environment' } },
  { label: 'front camera', constraints: { facingMode: 'user' } }
]
const constraintOptions = ref(defaultConstraintOptions)

async function onCameraReady() {
  const devices = await navigator.mediaDevices.enumerateDevices()
  const videoDevices = devices.filter(({ kind }) => kind === 'videoinput')

  constraintOptions.value = [
    ...defaultConstraintOptions,
    ...videoDevices.map(({ deviceId, label }) => ({
      label: `${label} (ID: ${deviceId})`,
      constraints: { deviceId }
    }))
  ]

  error.value = ''
}

/*** track functons ***/
function paintOutline(detectedCodes, ctx) {
  for (const detectedCode of detectedCodes) {
    const [firstPoint, ...otherPoints] = detectedCode.cornerPoints
    ctx.strokeStyle = 'red'
    ctx.beginPath()
    ctx.moveTo(firstPoint.x, firstPoint.y)
    for (const { x, y } of otherPoints) {
      ctx.lineTo(x, y)
    }
    ctx.lineTo(firstPoint.x, firstPoint.y)
    ctx.closePath()
    ctx.stroke()
  }
}
function paintBoundingBox(detectedCodes, ctx) {
  for (const detectedCode of detectedCodes) {
    const {
      boundingBox: { x, y, width, height }
    } = detectedCode
    ctx.lineWidth = 2
    ctx.strokeStyle = '#007bff'
    ctx.strokeRect(x, y, width, height)
  }
}
function paintCenterText(detectedCodes, ctx) {
  for (const detectedCode of detectedCodes) {
    const { boundingBox, rawValue } = detectedCode

    const centerX = boundingBox.x + boundingBox.width / 2
    const centerY = boundingBox.y + boundingBox.height / 2

    const fontSize = Math.max(12, (50 * boundingBox.width) / ctx.canvas.width)

    ctx.font = `bold ${fontSize}px sans-serif`
    ctx.textAlign = 'center'

    ctx.lineWidth = 3
    ctx.strokeStyle = '#35495e'
    ctx.strokeText(detectedCode.rawValue, centerX, centerY)

    ctx.fillStyle = '#5cb984'
    ctx.fillText(rawValue, centerX, centerY)
  }
}
// const trackFunctionOptions = [
//   { text: 'centered text', value: paintCenterText },
//   { text: 'outline', value: paintOutline },
//   { text: 'nothing', value: undefined  },
//   { text: 'bounding box', value: paintBoundingBox }
// ]
const trackFunctionOptions = [
  { text: 'nothing (default)', value: undefined },
  { text: 'outline', value: paintOutline },
  { text: 'centered text', value: paintCenterText },
  { text: 'bounding box', value: paintBoundingBox }
]
const trackFunctionSelected = ref(trackFunctionOptions[1])

/*** barcode formats ***/

const barcodeFormats = ref({
  aztec: false,
  code_128: false,
  code_39: false,
  code_93: false,
  codabar: false,
  databar: false,
  databar_expanded: false,
  data_matrix: false,
  dx_film_edge: false,
  ean_13: false,
  ean_8: false,
  itf: false,
  maxi_code: false,
  micro_qr_code: false,
  pdf417: false,
  qr_code: true,
  rm_qr_code: false,
  upc_a: false,
  upc_e: false,
  linear_codes: false,
  matrix_codes: false
})
const selectedBarcodeFormats = computed(() => {
  return Object.keys(barcodeFormats.value).filter((format) => barcodeFormats.value[format])
})

/*** error handling ***/

const error = ref('')

function onError(err) {
  error.value = `[${err.name}]: `

  if (err.name === 'NotAllowedError') {
    error.value += '你需要开启相机权限'
  } else if (err.name === 'NotFoundError') {
    error.value += '该设备没有相机'
  } else if (err.name === 'NotSupportedError') {
    error.value += '需要安全上下文（HTTPS、本地主机）'
  } else if (err.name === 'NotReadableError') {
    error.value += '摄像头被占用？'
  } else if (err.name === 'OverconstrainedError') {
    error.value += '安装的摄像头不适合'
  } else if (err.name === 'StreamApiNotSupportedError') {
    error.value += '该浏览器不支持摄像头'
  } else if (err.name === 'InsecureContextError') {
    error.value +=
      '仅在安全环境下才允许访问摄像头。使用 HTTPS 而不是 HTTP.'
  } else {
    error.value += err.message
  }
}

/***  device button  ***/
/*** deal device ***/

function cata(){
	var resultString = result.value
	// var resultString="B-5F-001"  //测试数据
	
	uni.request({
		//新写接口post列表
	  // url: 'http://192.168.10.95:8082/fire/firefighting/infor',
		url: 'http://121.37.141.204:8082/fire/firefighting/infor',
	  method: 'POST',
	  data: {
			qrContent :resultString
		},
	  header: { 'content-type': 'application/json'},
	  success: (res) => {
	    // console.log('请求成功', res);
			fireid.value = res.data.toString();
			
			queryRecord();
		},
	  fail: (res) => {
	    console.log('请求失败', res);
		},
	});
}


	function queryRecord(){
		uni.request({
			// url:'http://192.168.10.95:8082/record/record/infor',
			url:'http://121.37.141.204:8082/record/record/infor',
		  method: 'POST', 
		  data: {
				"fireId":fireid.value
			},
		  header: { 
		    'content-type': 'application/json' 
		  },
		  success: (res) => {
				inforList.value = res.data;
				
				if(res.data.length == 0){
					resultinfo.value = "此设备无点检记录"
				}
				// console.log( inforList.value.length);
				// console.log(inforList.value)
		  },
		  fail: (err) => {
		    console.log('请求失败', err);
				uni.showModal({
					content:"查询失败，请重试"
				})
		  },
		 //  complete: () => {
		 //    console.log('请求完成');
			// },
		});
		
		
	}


function goCheck(){
	// window.location.href = 'fireH5/pages/index2/index2';
	uni.navigateTo({
		url: '/pages/index2/index2'
	});
	
}
</script>

<style scoped>
	.decode-result{
		margin: 25rpx;
	}
	
	.form-item {
	  margin-bottom: 20rpx;
	  display: flex;
	  align-items: center;
	}
	.boolean-choice {
	  margin-left: 10rpx;
	  display: flex;
	  align-items: center;
	}
	
	
.error {
  font-weight: bold;
  color: red;
}
.barcode-format-checkbox {
  margin-right: 10px;
  white-space: nowrap;
  display: inline-block;
}

	
.imageButton{
	margin: 10px;
}
.liclass{
	/* margin-left: -3rpx; */
}
	
.resultinfo{
	margin: 50rpx;
	font-size: 26px;
	justify-content: center;
	display: flex;
}
</style> 