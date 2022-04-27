<template>
	<div id="app">
		<div class="calendar-parent">
			<calendar-view
				:items="items"
				:show-date="showDate"
				:time-format-options="{ hour: 'numeric', minute: '2-digit' }"
				:enable-drag-drop="false"
				:disable-past="false"
				:disable-future="false"
				:show-times="false"
				display-period-uom="month"
				:display-period-count="1"
				:starting-day-of-week="1"
				:class="themeClasses"
				:current-period-label="title"
				:displayWeekNumbers="true"
				:enable-date-selection="true"
				@click-item="onClickItem"
			>
				<template #header="{ headerProps }">
					<calendar-view-header slot="header" :header-props="headerProps" @input="setShowDate" />
				</template>
			</calendar-view>
		</div>
		<div v-if="message" class="notification is-success">{{ message }}</div>
	</div>
</template>
<script>
import "../node_modules/vue-simple-calendar/dist/style.css"
import "../node_modules/vue-simple-calendar/static/css/default.css"
import "../node_modules/vue-simple-calendar/static/css/holidays-us.css"

import ICAL from "ical.js"

import { CalendarView, CalendarViewHeader, CalendarMath } from "vue-simple-calendar"

export default {
	name: "App",
	components: {
		CalendarView,
		CalendarViewHeader,
	},
	data() {
		return {
			/* Show the current month, and give it some fake items to show */
			title: "Calendar",
			showDate: this.thisMonth(1),
			message: "",

			items: [
				{
					id: "e0",
					startDate: "2021-01-14",
					endDate: "2021-01-20",
					title: "Multi-day item with a long title and times",
				},
			],
		}
	},
	computed: {
		userLocale() {
			return CalendarMath.getDefaultBrowserLocale
		},
		dayNames() {
			return CalendarMath.getFormattedWeekdayNames(this.userLocale, "long", 0)
		},
		themeClasses() {
			return {
				"theme-default": true,
			}
		},
	},
	mounted() {
		const urlParams = new URLSearchParams(window.location.search)
		const ics = urlParams.get("ics") || prompt("Please enter a URL to your ICS/ICAL file")
		this.title = urlParams.get("title") || prompt("Please enter a Title for your calendar")
		window.document.title = this.title
		const search = `?title=${this.title}&ics=${ics}`
		if (window.location.search != search) {
			window.location.search = search
		}
		this.tryFetch(ics)
			.then((data) => {
				const vcalendar = new ICAL.Component(ICAL.parse(data))
				this.items = Array.from(vcalendar.getAllSubcomponents("vevent"), (vevent, id) => {
					var event = new ICAL.Event(vevent)
					return {
						id: "event" + id,
						startDate: event.startDate.toString(),
						endDate: event.endDate.toString(),
						title: event.summary,
					}
				})
			})
	},

	methods: {
		thisMonth(d, h, m) {
			const t = new Date()
			return new Date(t.getFullYear(), t.getMonth(), d, h || 0, m || 0)
		},
		setShowDate(d) {
			this.showDate = d
		},
		onClickItem(e) {
			this.message = e.title
		},
		async fetch(url) {
			const response = await fetch(url);
			if(!response.ok) {
				throw response.status;
			}
			return await response.text()
		},
		async thingproxy(url) {
			return await this.fetch("https://thingproxy.freeboard.io/fetch/" + url)
		},
		async allorigins(url) {
			const data = await this.fetch("https://api.allorigins.win/get?url=" + url)
			return JSON.parse(data).contents;
		},
		async tryFetch(url) {
			const services = [this.thingproxy, this.allorigins];
			for (const service of services) {
				try {
					return await service(url)
				} catch(error) {
					console.log(error);
				}
			}
		}
	},
}
</script>

<style>
html,
body {
	height: 100%;
	margin: 0;
	background-color: #f7fcff;
}

#app {
	display: flex;
	flex-direction: column;
	font-family: Calibri, sans-serif;
	width: 95vw;
	min-width: 30rem;
	max-width: 100rem;
	min-height: 40rem;
	margin-left: auto;
	margin-right: auto;
}

.calendar-parent {
	display: flex;
	flex-direction: column;
	flex-grow: 1;
	overflow-x: hidden;
	overflow-y: hidden;
	max-height: 100vh;
	background-color: white;
}
</style>