<template>
	<v-calendar ref="calendar" :first-day-of-week="2" expanded v-model="selectedDate" :attributes='attrs'
		@update:pages="pageChange">
		<template #footer>
			<div class="w-full px-4 pb-3">
				<p class="bg-indigo-600 hover:bg-indigo-700 font-bold w-full px-3 py-1 rounded-md">
					Present: {{ presentData.length }} | Absent: {{ absentData.length }}
				</p>
			</div>

			<slot></slot>
		</template>
	</v-calendar>
</template>

<script setup>
import { createResource, createListResource, toast, FeatherIcon } from "frappe-ui"
import { computed, inject, ref, onMounted, onBeforeUnmount } from "vue"
const DOCTYPE = "Attendance"

const socket = inject("$socket")
const employee = inject("$employee")
const dayjs = inject("$dayjs")
const selectedDate = ref(new Date())
const startDate = ref(dayjs().startOf('month').format('YYYY-MM-DD'))
const endDate = ref(dayjs().endOf('month').format('YYYY-MM-DD'))
const attendance = createListResource({
	doctype: DOCTYPE,
	fields: ["status", "employee_name", "attendance_date", "employee", "late_entry", "early_exit"],
	filters: [
		["employee", "=", employee.data.name],
		["attendance_date", ">=", startDate.value],
		["attendance_date", "<=", endDate.value],
	],
	orderBy: "attendance_date desc",
	pageLength: 31,
})
attendance.reload()


const presentData = computed(() => {
	if (!attendance.data) return [];

	const presentDates = attendance.data
		.filter(att => att.status === 'Present')
		.map(att => att.attendance_date)
		.filter(date => date); // Filter out undefined values

	return presentDates;
});

const absentData = computed(() => {
	if (!attendance.data) return [];

	const presentDates = attendance.data
		.filter(att => att.status === 'Absent')
		.map(att => att.attendance_date)
		.filter(date => date); // Filter out undefined values

	return presentDates;
});

const attrs = computed(() => [
	{
		key: 'today',
		highlight: true,
		dates: new Date(),
	},
	{
		bar: 'green',
		dates: presentData.value,
	},
	{
		bar: 'red',
		dates: absentData.value,
	},
]);

function pageChange(page) {
	const startDateX = `${page[0].id}-01`
	const endDateX = `${page[0].id}-${dayjs(page[0].id).daysInMonth()}`
	if (startDateX !== startDate.value || endDateX !== endDate.value) {
		attendance.filters = [
			["employee", "=", employee.data.name],
			["attendance_date", ">=", startDateX],
			["attendance_date", "<=", endDateX],
		]
		startDate.value = startDateX
		endDate.value = endDateX
		attendance.reload()
	}
}

onMounted(() => {
	socket.emit("doctype_subscribe", DOCTYPE)
	socket.on("list_update", (data) => {
		if (data.doctype == DOCTYPE) {
			checkins.reload()
		}
	})
})


</script>