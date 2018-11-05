<template>
		<Page>
				<ActionBar title="Welcome to NativeScript-Vue!"/>
				<GridLayout columns="*" rows="*">
						<Label class="message" :text="msg" col="0" row="0"/>
						<Button col="1" row="0" text="Add task" @tap="onButtonTap" />
				</GridLayout>
		</Page>
</template>

<script>
	export default {
		methods: {
			onButtonTap: function(args) {
				console.log(process.env.METEO_API_KEY)
			}
		},
		data() {
			return {
				msg: 'Hello World!'
			}
		}
	}
</script>

<style scoped>
ActionBar {
	background-color: purple;
	color: #ffffff;
}

.message {
	vertical-align: center;
	text-align: center;
	font-size: 20;
	color: #333333;
}
</style>
