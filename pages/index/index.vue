<template>
	<button type="default" @click="add">新增</button>
	<view class="data">
		<view class="boxItem" v-for="(item, index) in data" :key="index">
			<view class="name">{{ item.name }}</view>
			<view class="old">{{ item.old }}</view>
			<view>
				<button @click="setUpdate(item)" type="primary" size="mini">修改</button>
				<button type="warn" size="mini" @click="dellibrary(item._id)">删除</button>
			</view>
		</view>
	</view>

	<uni-popup ref="inputDialogRef" type="share" @maskClick="cancel">
		<view style="background-color: white;padding: 50rpx 30rpx 20rpx;">
			<view style="margin-bottom: 20rpx;">{{ dialogTitle }}</view>
			<uni-forms ref="baseForm">
				<uni-forms-item label="姓名">
					<uni-easyinput v-model="inputData.name" placeholder="请输入姓名" />
				</uni-forms-item>
				<uni-forms-item label="年龄">
					<uni-easyinput v-model="inputData.old" placeholder="请输入年龄" />
				</uni-forms-item>
			</uni-forms>
			<view>
				<button type="primary" @click="submit">确认</button>
				<button style="margin-top: 10rpx;" @click="cancel">取消</button>
			</view>
		</view>
	</uni-popup>

	<uni-popup ref="messageRef" type="message">
		<uni-popup-message :type="messageData.msgType" :message="messageData.messageText"
			:duration="2000"></uni-popup-message>
	</uni-popup>
</template>

<script setup>
import { ref } from 'vue'
const data = ref()
const inputData = ref({}) //表单数据
const inputDialogRef = ref(null) // 弹窗组件ref
const messageRef = ref(null)
const dialogTitle = ref('') // 弹窗标题
const db = uniCloud.database();// 链接数据库
const messageData = ref({
	msgType: "success",
	messageText: '成功消息'
})

// 新增按钮
const add = () => {
	dialogTitle.value = '新增'
	inputDialogRef.value.open()
}
// 修改按钮
const setUpdate = (item) => {
	dialogTitle.value = '修改'
	inputDialogRef.value.open()
	inputData.value = { ...item } // 拷贝一份后赋值
}
// 取消按钮
const cancel = () => {
	dialogTitle.value = ''
	inputData.value = {}
	inputDialogRef.value.close()
}
// 确认按钮
const submit = () => {
	uni.showLoading({
		title: '提交中'
	})
	if (inputData.value._id) {
		setlibrary(inputData.value)
	} else {
		addlibrary(inputData.value)
	}
}

/**
 * 获取数据库
 */
const getlibrary = () => {
	uni.showLoading({
		title: '加载中'
	})
	db.collection("user").get().then(res => {
		data.value = res.result.data
		console.log(res.result.data)
		uni.hideLoading()
	})
}


/** 
 * 新增记录
 */

const todo = uniCloud.importObject('add')
async function addlibrary(data) {
	try {
		await todo.add(data).then(() => {
			messageToggle("success", "新增成功")
			getlibrary()
			cancel()
		})
	} catch (e) { console.log(e) };
}

/**
 * 修改记录
 */
const setlibrary = (data) => {
	const { _id, name, old } = data
	db.collection("user").doc(_id).update({ name, old }).then(() => {
		messageToggle('success', '修改成功')
		getlibrary()
		cancel()
	})
}

/**
 * 删除记录
 */
const dellibrary = (id) => {
	db.collection("user").doc(id).remove().then(() => {
		messageToggle('success', '删除成功')
		getlibrary()
		cancel()
	})
}

const messageToggle = (type, text) => {
	messageData.value.msgType = type
	messageData.value.messageText = text
	messageRef.value.open()
}


// 请求数据
getlibrary()


/**
 * 调用云函数
 */
/* uniCloud.callFunction({
	name: 'cloudfun01',
	data: {
		title: 'Hello CloudFun'
	},
	timeout: 100
}).then((res) => {
	console.log(res);
}) */

/**
 * 调用云对象
 */
/* const todo = uniCloud.importObject('cloudObj01')
try {
	const res = await todo.add(1, 2)
	console.log(res);
	
} catch (e) { console.log(e) }; */
</script>

<style lang="scss" scoped>
.content {
	display: flex;
	flex-direction: column;
	align-items: center;
	justify-content: center;
}

.data {
	margin-top: 60rpx;

	.boxItem {
		display: flex;
		justify-content: space-between;
		align-items: center;
	}
}

.input {
	border: 1px solid black;
	max-width: 200rpx;
}
</style>