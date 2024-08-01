<template>
	<v-calendar ref="calendar" :first-day-of-week="2" expanded v-model="selectedDate" :attributes='attrs'>
		<template #footer>
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
const attendanceData = ref([])
const attrs = ref([
	{
		key: 'today',
		highlight: true,
		dates: new Date(),
	},
]);
const attendance = createListResource({
	doctype: DOCTYPE,
	fields: ["status", "employee_name", "attendance_date", "employee", "late_entry", "early_exit"],
	filters: {
		employee: employee.data.name,
	},
	orderBy: "attendance_date desc",
})
attendance.reload()


const attData = computed(() => {
	if (!attendance.data) return {}
	console.log(attendance.data)
	return attendance.data;
})


onMounted(() => {
	socket.emit("doctype_subscribe", DOCTYPE)
	socket.on("list_update", (data) => {
		if (data.doctype == DOCTYPE) {
			checkins.reload()
		}
	})
})


</script>