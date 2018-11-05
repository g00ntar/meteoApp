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
	import axios from 'axios';

	const COORDINATES = '188,281';
	const TEMP_URL = `https://api.meteo.pl/api/v1/model/wrf/grid/d01_XLONG_XLAT/coordinates/${COORDINATES}/field/TSK/level/0/date/2018-11-02T18/forecast/`;
	const HEADERS = {
		Authorization: `Token ${process.env.METEO_API_KEY}`
	};

	const KELVIN = -273.15

	const PARSERS = {
		TEMP: function(data) {
			return Math.round((data+KELVIN)*10)/10
		} 
	}

	function parseData(responseData, parser) {
		return responseData.times.map((t,i)=>{
			return {
				time: t,
				data: parser(responseData.data[i])
			}
		})
	}

	export default {
		methods: {
			onButtonTap: function(args) {
				axios({
					method: 'POST',
					url: TEMP_URL,
					headers: HEADERS
				})
				.then(r=>r.data)
				.then(d=>parseData(d,PARSERS.TEMP))
				.then(console.log)
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
