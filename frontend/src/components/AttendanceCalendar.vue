<template>
    <div>
        <v-calendar v-model="selectedDate" :events="attendanceData" />
    </div>
</template>

<script setup>
import { createResource, createListResource, toast, FeatherIcon } from "frappe-ui"
const DOCTYPE = "Attendance"

const attendance = createListResource({
	doctype: DOCTYPE,
	fields: ["leave_type", "status", "employee_name", "attendance_date", "employee", "late_entry", "early_exit"],
	filters: {
		employee: employee.data.name,
	},
	orderBy: "time desc",
})
attendance.reload()


</script>