<template>
  <view>
<!-- 		<p>
      <select v-model="selectedConstraints">
        <option
          v-for="option in constraintOptions"
          :key="option.label"
          :value="option.constraints"
        >
          {{ option.label }}
        </option>
      </select>
    </p> -->

<!--    <p>
      <select v-model="trackFunctionSelected">
        <option
          v-for="option in trackFunctionOptions"
          :key="option.text"
          :value="option"
        >
          {{ option.text }}
        </option>
      </select>
    </p> -->

    <p class="error">{{ error }}</p>

    <p class="decode-result">
      扫描结果: <b>{{ result }}</b>
    </p>

    <view>
      <qrcode-stream
        :constraints="selectedConstraints"
        :track="trackFunctionSelected.value"
        :formats="selectedBarcodeFormats"
        @error="onError"
        @detect="onDetect"
        @camera-on="onCameraReady"
      />
    </view>
		<button @click="record" class="imageButton">已检查</button>
		<button @click="takePhoto" class="imageButton">拍照上传</button>
				

		</view>
</template>

<script setup lang="ts">
import { ref, computed, onMounted } from 'vue'	


/*** detection handling ***/
import { QrcodeStream, QrcodeDropZone, QrcodeCapture } from 'vue-qrcode-reader'


const result = ref('')
// onMounted(()=>{
// 	uni.showModal({
// 		content:result
// 	})
// })
const MyComponent = {
  components: {
    QrcodeStream,
    QrcodeDropZone,
    QrcodeCapture
  }
}
function onDetect(detectedCodes) {
  console.log(detectedCodes)
  result.value = JSON.stringify(detectedCodes.map((code) => code.rawValue)).slice(2, -2) ;
	// result.value = JSON.parse (detectedCodes.map((code) => code.rawValue))
	console.log(detectedCodes.map((code) => code.rawValue)[0])

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

	function takePhoto(){
		uni.chooseImage({
		count: 1, // 默认9
		sizeType: ['original', 'compressed'], // 可以指定是原图还是压缩图，默认二者都有
		sourceType: ['album', 'camera'], // 可以指定来源是相册还是相机，默认二者都有
		success: function (res) {
			uni.showToast({
				title: '上传中，请不要关闭！',
				icon: 'none',
				duration: 10000
			});
			// 返回选定照片的本地文件路径列表
			if (res.tempFilePaths && res.tempFilePaths.length > 0) {
				let imagePaths = res.tempFilePaths[0];
						// uni.showToast({
						// 	title: '照片上传:'+ imagePaths,
						// 	icon: 'none'
						// });
			uni.uploadFile({
				url: 'https://yanru.zicp.fun/common/upload',
				filePath: imagePaths, 	// 要上传文件资源的路径
				name: 'file', 	// 文件对应的key
				success: (uploadFileRes) => {
					uni.showToast({
						title: '上传中，请不要关闭！',
						icon: 'none'
					});
					console.log(uploadFileRes.data);
					console.log(imagePaths)
					// 处理上传成功后的结果
					var image = JSON.parse(uploadFileRes.data).fileName
					uni.request({
						url: 'https://yanru.zicp.fun/fire/firefighting/uni',
						method: 'POST',
						data: {
							// 'qrContent': "第一个第一个",
							'qrContent': result.value,
							'image': image
						},
						header: {
							'Content-Type': 'application/json'
						},
						success: (res) => {
							console.log(res.data);
							uni.showToast({
								title: '上传成功',
								icon: 'none'
							});
						},
						fail: (res) => {
							console.log(res);
							uni.showToast({
								title: '照片上传失败'+ res,
								icon: 'none'
							});
						}
					});

					uni.navigateTo({
						url: '/pages/index/uplaod'
					});
				},
				fail: (uploadFileErr) => {
					// 处理上传失败的错误信息
					console.log(uploadFileErr);
					uni.showToast({
						title: '上传失败'+ JSON.stringify(uploadFileErr),
						icon: 'none'
					});
				}
			});
	
		} else {
			uni.showToast({
				title: '未选取图片',
				icon: 'none'
			});
		}
		
		
		}
	});

	}
	
	var resultString  =  result.value
	function record(){
		console.log(resultString)
		console.log(result)
		
		var formGet={}
		// formGet["qrContent"] = "第三的二维码内容灭火器高品高"
		formGet["qrContent"] = result.value
		console.log(formGet["qrContent"])
		var qrContent="第三的二维码内容灭火器高品高"
		// qrContent=JSON.stringify(qrContent)
		uni.showModal({
			content:formGet["qrContent"]
		})
		uni.request({
			//新写接口post列表
		  url: 'https://yanru.zicp.fun/fire/firefighting/getlist',
		  method: 'GET',
			// data: formGet,
		  data: {
				qrContent
			},
			// header: { 'content-type': 'application/x-www-form-urlencoded;' },
		  header: { 'content-type': 'application/json'},
		  success: (res) => {
		    console.log('请求成功', res);
				// console.log(res.data.filter(item => item.qrContent =="第三的二维码内容灭火器高品高").length); 
				
				//把消防设备res列表的检查字段等新增到检查流水表单 
				let date = new Date();
				let year = date.getFullYear();
				let month = date.getMonth() + 1;
				let day = date.getDate();
				let hour = date.getHours();
				let min = date.getMinutes();
				let seconds = date.getSeconds();
				var form={}
				form["fireId"] = res.data[0].fireId
				form["deviceName"] = res.data[0].deviceName
				form["deviceModel"] = res.data[0].deviceModel
				form["locate"] = res.data[0].locate
				form["checkRecords"] = year+"-"+month+"-"+day+" "+hour+":"+ min+":"+ seconds
				form["pointCheck"] = "已点检"
				uni.request({
				  url: 'https://yanru.zicp.fun/record/record/add',
				  method: 'POST', 
				  data: form,
				  header: { 
				    'content-type': 'application/json' 
				  },
				  success: (res) => {
				    console.log('新增成功', res.data);
							
						var form2={}
						form2["pointCheck"] = "是"
						form2["fireId"] = form["fireId"]
						uni.request({
							url: 'https://yanru.zicp.fun/fire/firefighting/update',
							method: 'POST', 
							data: form2,
							header: { 
								'content-type': 'application/json' 
							},
							success: (res) => {
								console.log('更新成功', res.data);
								// uni.showModal({
								// 	content:"更新成功"
								// })
							},
							fail: (err) => {
								console.log('更新失败', err);
								// uni.showModal({
								// 	content:"更新失败，请重试"
								// })
							}
						});	
						
				  },
				  fail: (err) => {
				    console.log('请求失败', err);
						// uni.showModal({
						// 	content:"新增失败，请重试"
						// })
				  },
				  complete: () => {
				    console.log('请求完成');
					},
				});	
				

		  },
		  fail: (err) => {
		    console.log('请求失败', err);
				uni.showModal({
					content:"点检失败，请重试"
				})
		  },
		  complete: () => {
		    console.log('请求完成');
				uni.showModal({
					content:"点检成功"
				})
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

	
.imageButton{
	margin: 10px;
}
</style>