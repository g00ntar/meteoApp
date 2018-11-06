<template>
		<Page>
			<ActionBar title="Meteo App"/>
			<TabView>
				<TabViewItem title="Settings">
					<StackLayout >
							<Label class="message" :text="msg"/>
							<DatePicker v-model="selectedDate" />
							<ListPicker v-model="selectedTime" :items="listOfTimes" />
							<Button text="Add task" @tap="onButtonTap" />
					</StackLayout>
				</TabViewItem>
				<TabViewItem title="Table">
					<StackLayout>
						<Label class="message" text="Table"/>
						<ListView for="item in dataSource"> 
							<v-template>
								<FlexboxLayout>
									<Label class="table-item" :text="item.time" width="48%"/> 
									<Label class="table-item" :text="item.data" width="48%"/>
								</FlexboxLayout> 
							</v-template>
						</ListView>
					</StackLayout>
				</TabViewItem>
				<TabViewItem title="Chart">
					<StackLayout>
						<Label class="message" text="Chart"/>
						<RadCartesianChart>
							<LineSeries
								v-tkCartesianSeries 
								:items="dataSource"
								categoryProperty="time"
								valueProperty="data"
							/>
							<CategoricalAxis v-tkCartesianHorizontalAxis />
							<LinearAxis v-tkCartesianVerticalAxis />
						</RadCartesianChart>
					</StackLayout>
				</TabViewItem>
			</TabView>
		</Page>
</template>

<script>
	import {ObservableArray} from "data/observable-array"

	import axios from 'axios';
	import { format } from 'date-fns/fp'

	const COORDINATES = '188,281';
	const METEO_API_URL = 'https://api.meteo.pl/api/v1/'
	const HEADERS = {
		Authorization: `Token ${process.env.METEO_API_KEY}`
	};

	const KELVIN = -273.15

	const DATATYPE = {
		DEFAULT: {
			parser: function(data){
				return data
			}
		},
		TIME: {
			parser: function(timeString){
				return timeString.split(':')[0]
			}
		},
		TEMP: {
			name: 'temp',
			field: 'T2',
			parser: function(data) {
				return Math.round((data+KELVIN)*10)/10
			},
		},
		HUMIDITY:{
			name: 'humidity',
			field: '',
		}
	}

	const formatDate = format('yyyy-MM-dd');

	function dateFormatter(date, time = '00') {
		return `${formatDate(date)}T${time}`
	}

	function urlBuilder(
		date = dateFormatter(new Date()),
		field = 'T2',
		coordinates = COORDINATES,
		grid = 'd01_XLONG_XLAT',
		model = 'wrf',
		level = 0,
	){
		return ["model", "grid", "coordinates", "field", "level", "date"]
				.reduce(function (accumulator,elementName){
					return accumulator + `${elementName}/${eval(elementName)}/`
				}, METEO_API_URL) + 'forecast/';
	}

	

	function parseData(responseData, parser) {
		return responseData.times.map((t,i)=>{
			return {
				time: PARSERS.TIME(t),
				data: parser(responseData.data[i])
			}
		})
	}

	export default {
		methods: {
			onButtonTap: function(args) {
					let url = urlBuilder(dateFormatter(this.selectedDate, this.listOfTimes[this.selectedTime])
				);
				axios({
					method: 'POST',
					url: url,
					headers: HEADERS
				})
				.then(r=>r.data)
				.then(d=>parseData(d,PARSERS.TEMP))
				.then(d=>{console.log(d); return d})
				.then(d=>{this.dataSource=d})
				.catch(console.error)
			}
		},
		data() {
			return {
				msg: 'Choose date and time',
				selectedDate: new Date(),
				selectedTime: 0,
				listOfTimes: [
					"00",
					"06",
					"12",
					"18"
			
				],
				dataSource: []

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

.table-item {
	text-align: center;
}
</style>
