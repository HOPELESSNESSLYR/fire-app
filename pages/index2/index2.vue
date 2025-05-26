<template>
  <view>
    <p class="error">{{ error }}</p>

    <p class="decode-result">
      扫描结果: <b>{{ result }}</b>
    </p>
		
		<p class="decode-result">
		  设备分组: <b>{{ selectedCategory }}</b>
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

			
			<view class="new">
					
			    <view v-if="selectedCategory">	
						<view v-for="(item, index) in formItems" :key="index" class="form-item">

						  <view style="width: 450rpx;">
						    <label>{{ item.label }}</label>
						  </view>
						
							<radio-group :v-model="item.isSelected" @change="getIsSelect(item,index)">
								<label v-for="(item,index) in redioItem" :key="index">
			
									<label :for="item.name">
										<text>{{item.value+' '}} </text>
									</label>
									
									<radio :id="item.name" :value="item.name" :checked="item.checked" v-model="item.checked"></radio>
								</label>
								
							</radio-group>
							
						</view>
			    </view>
			  </view>
	
		
		<button @click="record" class="imageButton">已检查</button>
		<button @click="takePhoto" class="imageButton">拍照上传</button>
		</view>
</template>

<script setup lang="ts">
import { ref, computed, onMounted } from 'vue'	

/*** detection handling ***/
import { QrcodeStream, QrcodeDropZone, QrcodeCapture } from 'vue-qrcode-reader'


/*** select device ***/
const fireid = ref('')
// const result = ref('第5个第5个')			//测试数据
const result = ref('')
let qrcode = ref(true)
const	selectedCategory = ref('');
const	categories = ref(['消火栓', '灭火器', '水泵房', '阀组间']);
const redioItem = ref([
	{name: "Y", value: "Y",checked: ''},
	{name: "N", value: "N"}
])
const formItems = ref([])
const formItems1 = ref([
        { label: "水带", value: "水带", isSelected: 'true' },
        { label: "水枪", value: "水枪", isSelected: 'true' },
        { label: "卷盘", value: "卷盘", isSelected: 'true' },
        { label: "开关", value: "开关", isSelected: 'true' },
        { label: "外观", value: "外观", isSelected: 'true' },
        { label: "按钮", value: "按钮", isSelected: 'true' },
      ])
const formItems2 = ref([
			{ label: "压力", value: "压力", isSelected: 'true' },
			{ label: "喷嘴", value: "喷嘴", isSelected: 'true' },
			{ label: "瓶体", value: "瓶体", isSelected: 'true' },
			{ label: "压把", value: "压把", isSelected: 'true' },
			{ label: "保险销", value: "保险销", isSelected: 'true' },
			{ label: "有效期", value: "有效期", isSelected: 'true' },
		])
const formItems3 = ref([
			{ label: "压力", value: "压力", isSelected: 'true' },
			{ label: "消防供电", value: "消防供电", isSelected: 'true' },
			{ label: "水泵", value: "水泵", isSelected: 'true' },
			{ label: "自动状态", value: "自动状态", isSelected: 'true' },
			{ label: "污水泵", value: "污水泵", isSelected: 'true' },
			{ label: "现场环境", value: "现场环境", isSelected: 'true' },
		])
const formItems4 = ref([
			{ label: "报警阀组", value: "报警阀组", isSelected: 'true' },
			{ label: "压力表", value: "压力表", isSelected: 'true' },
			{ label: "水力警铃", value: "水力警铃", isSelected: 'true' },
			{ label: "管道", value: "管道", isSelected: 'true' },
			{ label: "阀门", value: "阀门", isSelected: 'true' },
			{ label: "现场环境", value: "现场环境", isSelected: 'true' },
		])

// onMounted(()=>{
// 	// cata()
// 	ph()
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


/*** deal device ***/

// function onCategoryChange(e) {
// 	// console.log('picker发送选择改变，携带值为', categories.value[e.detail.value])
//       selectedCategory.value = categories.value[e.detail.value];
//       updateOptions();
// 			// console.log(JSON.stringify(formItems.value)) 
//     }
function cata(){
	var resultString = result.value
	// var resultString="第一个第一个"  //测试数据
	
	uni.request({
		//新写接口post列表
	  // url: 'https://yanru.zicp.fun/fire/firefighting/getlist',
		url: 'http://121.37.141.204:8082/fire/firefighting/getlist',
	  method: 'GET',
	  data: {
			qrContent :resultString
		},
	  header: { 'content-type': 'application/json'},
	  success: (res) => {
	    console.log('请求成功', res);
			selectedCategory.value = res.data[0].deviceGroup;
			fireid.value = res.data[0].fireId;
			console.log(fireid.value)
			updateOptions();
				// uni.showModal({
				// 	content:selectedCategory.value
				// })
		},
	  fail: (res) => {
	    console.log('请求失败', res);
		},
	});
}

function updateOptions() {
      if (selectedCategory.value === '消火栓') {
				formItems.value=formItems1.value

      } else if (selectedCategory.value === '灭火器') {
				formItems.value=formItems2.value

      } else if (selectedCategory.value === '水泵房') {
				formItems.value=formItems3.value

      } else if (selectedCategory.value === '阀组间') {
				formItems.value=formItems4.value
      }
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
			uni.uploadFile({
				// url: 'https://yanru.zicp.fun/common/upload',
				url: 'http://121.37.141.204:8082/common/upload',
				// url: 'http://127.0.0.1:8082/common/upload',		//测试
				filePath: imagePaths, 	// 要上传文件资源的路径
				name: 'file', 	// 文件对应的key
				success: (uploadFileRes) => {
					// uni.showToast({
					// 	title: '上传中，请不要关闭！',
					// 	icon: 'none'
					// });
					// console.log(uploadFileRes.data);
					// console.log(imagePaths)
					// 处理上传成功后的结果
					var image = JSON.parse(uploadFileRes.data).fileName
					uni.request({
						// url: 'https://yanru.zicp.fun/fire/firefighting/uni',
						url: 'http://121.37.141.204:8082/fire/firefighting/uni',
						// url: 'http://127.0.0.1:8082/fire/firefighting/uni',		//测试
						method: 'POST',
						data: {
							// 'qrContent': "第5个第5个",		//测试数据
							'qrContent': result.value,
							'image': image
						},
						header: {
							'Content-Type': 'application/json'
						},
						success: (res) => {
							console.log(res.data);
							
									var form4={}
									form4["fireId"] = fireid.value
									form4["image"] = image
									uni.request({
										// url: 'https://yanru.zicp.fun/record/record/uniphoto',
										url: 'http://121.37.141.204:8082/record/record/uniphoto',
										// url: 'http://127.0.0.1:8082/record/record/uniphoto', //测试
										method: 'POST',
										data: form4,
										// data:{																					//测试数据
										// 	"fireId": 5,
										// 	"image": image
										// },
										header: {
											'Content-Type': 'application/json'
										},
										success: (r) => {
											console.log(r.data);
															
											uni.navigateTo({
												url: '/pages/index/uplaod'
											});
											
										},
										fail: (r) => {
											console.log(r);
											uni.showToast({
												title: '上传失败'+ r,
											});
										}
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
	
	// var resultString  =  result.value
	function record(){
		var resultString  =  result.value
		// console.log("formItems.value"+JSON.stringify(formItems.value)) 
		const	postData = ref([]);
		let	falseNo = 0;
		for(var i=0;i<6;i++){	
			if(formItems.value[i].isSelected==false){
				falseNo++;
				postData.value.push(formItems.value[i].value)
				console.log('postData.value'+JSON.stringify(postData.value))
				// console.log("falseNo"+falseNo)
			}
		}
		
		console.log(resultString)
		

		// var resultString="第三的二维码内容灭火器高品高"  //测试数据
		
		uni.request({
			//新写接口post列表
		  // url: 'https://yanru.zicp.fun/fire/firefighting/getlist',
			url: 'http://121.37.141.204:8082/fire/firefighting/getlist',
		  method: 'GET',
		  data: {
				qrContent :resultString
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

				if(falseNo==0){
					form["qualified"] = "合格"
				}else{
					form["qualified"] = "不合格"
					form["feedback"] = postData.value[0] +"不正常"
					for(var i=1;i<falseNo;i++){
						form["feedback"] = postData.value[i]+'、'+form["feedback"]
					}
				}				

	
				uni.request({
				  // url: 'https://yanru.zicp.fun/record/record/add',
					url:'http://121.37.141.204:8082/record/record/add',
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
							// url: 'https://yanru.zicp.fun/fire/firefighting/update',
							url: 'http://121.37.141.204:8082/fire/firefighting/update',
							method: 'POST', 
							data: form2,
							header: { 
								'content-type': 'application/json' 
							},
							success: (res) => {
								// console.log('更新成功', res.data);
								// uni.showModal({
								// 	content:"更新成功"
								// })
								uni.showModal({
									content:"检查成功"
								})
							},
							fail: (err) => {
								console.log('更新失败', err);
								uni.showModal({
									content:"更新失败，请重试"
								})
							}
						});	
						
				  },
				  fail: (err) => {
				    console.log('请求失败', err);
						uni.showModal({
							content:"新增失败，请重试"
						})
				  },
				  complete: () => {
				    console.log('请求完成');
					},
				});	
				
		  },
		  fail: (err) => {
		    console.log('请求失败', err);
				uni.showModal({
					content:"检查失败，后台无此设备"
				})
		  },
		  complete: () => {
		    // console.log('请求完成');
				// uni.showModal({
				// 	content:"点检成功"
				// })
		  }
		});
			
		// 重置选项
		selectedCategory.value='';
		formItems.value=formItems.value;
	}
	function getIsSelect(item,index){
		formItems.value[index].isSelected = !formItems.value[index].isSelected;
		formItems.value[index].isSelected = !item.value;

		// console.log(JSON.stringify(formItems.value))
		// console.log(item.value)        
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
.new{
	margin-left: 35rpx;
	font-size: 34rpx;
}
.picker{
	margin-bottom: 20rpx;
}
</style>