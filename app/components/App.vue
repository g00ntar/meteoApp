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
				<TabViewItem title="Output">
					<!-- <Label class="message" :text="msg"/> -->
					<RadCartesianChart>
						<LineSeries v-tkCartesianSeries :items="categoricalSource" categoryProperty="Country" valueProperty="Amount" />
						<CategoricalAxis v-tkCartesianHorizontalAxis />
						<LinearAxis v-tkCartesianVerticalAxis />
					</RadCartesianChart>
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

	const PARSERS = {
		TEMP: function(data) {
			return Math.round((data+KELVIN)*10)/10
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
				time: t,
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
				.then(console.log)
				.catch(console.error)
			}
		},
		created() {
        this.categoricalSource = new ObservableArray([
            { Country: "Germany", Amount: 15, SecondVal: 14, ThirdVal: 24, Impact: 0, Year: 0 },
            { Country: "France", Amount: 13, SecondVal: 23, ThirdVal: 25, Impact: 0, Year: 0 },
            { Country: "Bulgaria", Amount: 24, SecondVal: 17, ThirdVal: 23, Impact: 0, Year: 0 },
            { Country: "Spain", Amount: 11, SecondVal: 19, ThirdVal: 24, Impact: 0, Year: 0 },
            { Country: "USA", Amount: 18, SecondVal: 8, ThirdVal: 21, Impact: 0, Year: 0 }
        ]);
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
				categoricalSource: ""

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
