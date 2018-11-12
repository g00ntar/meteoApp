<template>
		<Page>
			<ActionBar title="Meteo App"/>
			<TabView>
				<TabViewItem title="Settings">
					<StackLayout >
							<Label class="message" :text="msg"/>
							<DatePicker v-model="selectedDate" />
							<ListPicker v-model="selectedTime" :items="listOfTimes" />
							<Button :text="coordinates !== null ? 'Pobierz dane' : 'Oczekiwanie na lokalizację'" :isEnabled="coordinates !== null" @tap="onButtonTap" />
					</StackLayout>
				</TabViewItem>
				<TabViewItem title="Table">
					<StackLayout>
						<Label class="message" text="Table"/>
						<ListView for="item in dataSource" @itemTap="onItemTap"> 
							<v-template>
								<FlexboxLayout>
									<Label class="table-item table-item--time" :text="new Date(item.time)" /> 
									<Label class="table-item table-item--temperature" :text="item.TEMPERATURE" />
									<Label class="table-item table-item--pressure" :text="item.PRESSURE" />
									<Label class="table-item table-item--humidity" :text="item.HUMIDITY" />
								</FlexboxLayout> 
							</v-template>
						</ListView>
					</StackLayout>
				</TabViewItem>
				<TabViewItem title="Chart">
					<FlexboxLayout flexDirection="column">
						<StackLayout>
							<Label class="message" text="Temperatura"/>
							<RadCartesianChart>
								<LineSeries
									v-tkCartesianSeries 
									:items="dataSource"
									categoryProperty="time"
									valueProperty="TEMPERATURE"
								/>
								<DateTimeContinuousAxis dateFormat="hh" v-tkCartesianHorizontalAxis />
								<LinearAxis v-tkCartesianVerticalAxis />
							</RadCartesianChart>
						</StackLayout>
						<StackLayout>
							<Label class="message" text="Ciśnienie"/>
							<RadCartesianChart>
								<LineSeries
									v-tkCartesianSeries 
									:items="dataSource"
									categoryProperty="time"
									valueProperty="PRESSURE"
								/>
								<DateTimeContinuousAxis dateFormat="hh" v-tkCartesianHorizontalAxis />
								<LinearAxis v-tkCartesianVerticalAxis />
							</RadCartesianChart>
						</StackLayout>
						<StackLayout>
							<Label class="message" text="Wilgotność"/>
							<RadCartesianChart>
								<LineSeries
									v-tkCartesianSeries 
									:items="dataSource"
									categoryProperty="time"
									valueProperty="HUMIDITY"
								/>
								<DateTimeContinuousAxis dateFormat="hh" v-tkCartesianHorizontalAxis />
								<LinearAxis v-tkCartesianVerticalAxis />
							</RadCartesianChart>
						</StackLayout>
					</FlexboxLayout> 
				</TabViewItem>
				<TabViewItem title="Map">
					<StackLayout>

					</StackLayout>
				</TabViewItem>
			</TabView>
		</Page>
</template>

<script>
	// import {ObservableArray} from "data/observable-array"

	import axios from 'axios';
	import { format, getTime, addHours } from 'date-fns'

	// import * as geolocation from 'nativescript-geolocation';
	var geolocation = require("nativescript-geolocation");


	const ROWCOL = '188,281';
	const METEO_API_URL = 'https://api.meteo.pl/api/v1/'
	const HEADERS = {
		Authorization: `Token ${process.env.METEO_API_KEY}`
	};

	const GRID = 'd01_XLONG_XLAT';
	const MODEL = 'wrf';
	const LEVEL = 0;

	const KELVIN = -273.15

	const DATA_PARSERS = {
		DEFAULT: {
			parser: function(data){
				return data
			}
		},
		TIME: {
			parser: function(dataTimeString){
				// return timeString.split(':')[0].split('-')[2]
				let dataTimeArray = dataTimeString.split(':')[0].split('T')
				return getTime(addHours(new Date(dataTimeArray[0]), parseInt(dataTimeArray[1])))
			}
		},
		TEMPERATURE: {
			name: 'Temperatura powietrza',
			field: 'T2',
			unit: '℃',
			parser: function(data) {
				return Math.round((data+KELVIN)*10)/10
			},
		},
		SKIN_TEMPERATURE:{
			name: 'Temperatura odczuwalna',
			filed: 'TSK',
			unit: '℃',
			parser: function(data) {
				return Math.round((data+KELVIN)*10)/10
			},
		},
		HUMIDITY:{
			name: 'Wilgotność',
			field: 'Q2',
			unit: '%',
			parser: function(data){
				return Math.round(data*1000)/10
			}
		},
		PRESSURE: {
			name: 'Ciśnienie',
			field: 'PSFC',
			unit: 'hPa',
			parser: function(data){
				return Math.round(data/100)
			}
		},
		PRECIPITATION: {
			name: 'Opady',
			require: ['RANNIC', 'RANIC'],
			parser: function(data) {
				return data[0] + data[1]
			}
		},
		RANNIC: {
			field: 'RANNIC'
		},
		RANIC: {
			field: 'RANIC'
		}
	}

	const DEFAULT_LIST = ['TEMPERATURE', 'PRESSURE','HUMIDITY']

	function requestDateFormatter(date, time = '00') {
		return `${format(date,'yyyy-MM-dd')}T${time}`
	}

	function urlBuilder(options){
		options = Object.assign(
			{},{
				field: 'T2',
				coordinates: ROWCOL,
				date: requestDateFormatter(new Date()),
				grid: GRID,
				model: MODEL,
				level: LEVEL
			},
			options
		)

		// console.log(options)

		return ["model", "grid", "coordinates", "field", "level", "date"]
				.reduce(function (accumulator,elementName){
					return accumulator + `${elementName}/${options[elementName]}/`
				},'/') + 'forecast/';
	}

	function parseData(responseData, dataType) {
		return responseData.times.map((t,i)=>{
			let obj = {time: DATA_PARSERS.TIME.parser(t)};
			obj[dataType] = DATA_PARSERS[dataType].parser(responseData.data[i])
			return obj
		})
	}

	const meteoApi = axios.create({
		baseURL: METEO_API_URL,
		method: 'POST',
		headers: HEADERS
	});

	
	
	export default {
		methods: {
			onButtonTap: function(args) {

				let urlList = DEFAULT_LIST.map((dataType)=>{
					return urlBuilder({field: DATA_PARSERS[dataType].field, date: requestDateFormatter(this.selectedDate, this.listOfTimes[this.selectedTime])})
				})

				axios.all(urlList.map(url=>meteoApi({url})))
				.then(axios.spread(function (...Args) {
					let responseData = [];
					Args.forEach(response => {
						let field = response.config.url.split('/')[12]
						let dataType = Object.keys(DATA_PARSERS).find(key=>DATA_PARSERS[key].field == field);				
						parseData(response.data, dataType).forEach((dataEntry, index)=>{
							responseData[index] = Object.assign({},responseData[index], dataEntry)
						})
					});
					return responseData;
				}))
				.then(responseData=>{this.dataSource=responseData})
				.catch(console.error)
			},
			onItemTap: function (event) {
				console.log(event.item)
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
				dataSource: [],
				coordinates: null
			}
		},
		created: function(){
			 geolocation
			 .isEnabled()
			 .then(isEnabled=>{
				if (!isEnabled) {
					geolocation
					.enableLocationRequest()
					.then(event=>{
						console.log(event)
					})
					.catch(error=>{
						console.log("Error enableLocationRequest: " + (error.message || error));
					});
				}
			})
			.then(()=>{
				geolocation
				.getCurrentLocation()
				.then(location=>{
					this.coordinates = location
				}).catch(error=>{
					console.log("Error getCurrentLocation: " + (error.message || error));
				})
			})
			.catch(error=>{
				console.log("Error isEnabled: " + (error.message || error));
			});
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
	flex-basis: 1;
	flex-grow: 1;
}

.table-item--time {
	flex-shrink: 1;
}
</style>
