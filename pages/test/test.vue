<template>
  <div>

    <p class="error">{{ error }}</p>

    <p class="decode-result">
      Last result: <b>{{ result }}</b>
    </p>

    <div>
      <qrcode-stream
        :constraints="selectedConstraints"
        :track="trackFunctionSelected.value"
        :formats="selectedBarcodeFormats"
        @error="onError"
        @detect="onDetect"
        @camera-on="onCameraReady"
      />
    </div>
		<button @click="record" class="imageButton">已检查</button>
  </div>
</template>

<script setup lang="ts">
import { ref, computed ,onMounted} from 'vue'
import { QrcodeStream,QrcodeDropZone, QrcodeCapture } from 'vue-qrcode-reader'

/*** detection handling ***/

const result = ref('')

function onDetect(detectedCodes) {
  console.log(detectedCodes)
  result.value = JSON.stringify(detectedCodes.map((code) => code.rawValue))
	console.log(detectedCodes.map((code) => code.rawValue))
	// result.value = JSON.parse(detectedCodes.map((code) => code.rawValue))
}

/*** select camera ***/

const selectedConstraints = ref({ facingMode: 'environment' })
const defaultConstraintOptions = [
  { label: 'rear camera', constraints: { facingMode: 'environment' } },
  { label: 'front camera', constraints: { facingMode: 'user' } }
]
const constraintOptions = ref(defaultConstraintOptions)

async function onCameraReady() {
  // NOTE: on iOS we can't invoke `enumerateDevices` before the user has given
  // camera access permission. `QrcodeStream` internally takes care of
  // requesting the permissions. The `camera-on` event should guarantee that this
  // has happened.
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
    error.value += 'you need to grant camera access permission'
  } else if (err.name === 'NotFoundError') {
    error.value += 'no camera on this device'
  } else if (err.name === 'NotSupportedError') {
    error.value += 'secure context required (HTTPS, localhost)'
  } else if (err.name === 'NotReadableError') {
    error.value += 'is the camera already in use?'
  } else if (err.name === 'OverconstrainedError') {
    error.value += 'installed cameras are not suitable'
  } else if (err.name === 'StreamApiNotSupportedError') {
    error.value += 'Stream API is not supported in this browser'
  } else if (err.name === 'InsecureContextError') {
    error.value +=
      'Camera access is only permitted in secure context. Use HTTPS or localhost rather than HTTP.'
  } else {
    error.value += err.message
  }
}


		var resultString  =  result.value
		var formGet={}
		formGet["qrContent"] = "第三的二维码内容灭火器高品高"
		// formGet["qrContent"] = result.value
		console.log(formGet["qrContent"])
		
		uni.showModal({
			content:formGet["qrContent"]
		})

	
	function record(){
		console.log(resultString)
		console.log(result)
		uni.request({
			//新写接口post列表
		  url: 'https://huade.picp.vip/fire/firefighting/getlist',
		  method: 'POST',
		  data: formGet,
		  header: {
		    'content-type': 'application/json',
				'Access-Control-Allow-Origin':'*'
		  },
		  success: (res) => {
		    console.log('请求成功', res.data);
				//把消防设备res列表的检查字段等新增到检查流水表单 
				let date = new Date();
				let year = date.getFullYear();
				let month = date.getMonth() + 1;
				let day = date.getDate();
				let hour = date.getHours();
				let min = date.getMinutes();
				// let seconds = date.getSeconds();
				var form={}
				form["fireId"] = res.data[0].fireId
				form["deviceName"] = res.data[0].deviceName
				form["deviceModel"] = res.data[0].deviceModel
				form["locate"] = res.data[0].locate
				form["checkRecords"] = year+"-"+month+"-"+day+" "+hour+":"+ min
				form["pointCheck"] = "已点检"
				uni.request({
				  url: 'https://huade.picp.vip/record/record',
				  method: 'POST', 
				  data: form,
				  header: { 
				    'content-type': 'application/json' 
				  },
				  success: (res) => {
				    console.log('请求成功', res.data);
						uni.showModal({
							content:"新增成功"
						})
				  },
				  fail: (err) => {
				    console.log('请求失败', err);
						uni.showModal({
							content:"新增失败，请重试"
						})
				  },
				  complete: () => {
				    console.log('请求完成');
				  }
				});
				
		  },
		  fail: (err) => {
		    console.log('请求失败', err);
				uni.showModal({
					content:"更新检查记录失败，请重试"
				})
		  },
		  complete: () => {
		    console.log('请求完成');
		  }
		});
	}
	

</script>

<style scoped>
.error {
  font-weight: bold;
  color: red;
}
.barcode-format-checkbox {
  margin-right: 10px;
  white-space: nowrap;
  display: inline-block;
}
</style>