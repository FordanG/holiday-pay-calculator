<template>
  <div>
    <p>Current Day: {{ currentDay }}</p>

    <div class="p-field">
      <label for="monthlyPay">Monthly Pay</label>
      <InputNumber
        id="monthlyPay"
        v-model="monthlyPay"
        mode="currency"
        currency="PHP"
      />
    </div>
    <div class="p-field">
      <label for="dailyPay">Daily Pay</label>
      <InputNumber
        id="dailyPay"
        v-model="dailyPay"
        mode="currency"
        currency="PHP"
      />
    </div>
    <div class="p-field">
      <label for="hourlyPay">Hourly Pay</label>
      <InputNumber
        id="hourlyPay"
        v-model="hourlyPay"
        mode="currency"
        currency="PHP"
      />
    </div>
    <div class="p-field-checkbox">
      <Checkbox id="restDay" v-model="isRestDay" :binary="true" />
      <label for="restDay">Rest Day</label>
    </div>
    <div class="p-field">
      <label for="holidayType">Holiday</label>
      <Dropdown
        id="holidayType"
        v-model="selectedHoliday"
        :options="holidays"
        optionLabel="name"
      />
    </div>
    <div class="p-field">
      <label for="hoursWorked">Hours Worked</label>
      <InputNumber id="hoursWorked" v-model="hoursWorked" />
    </div>
    <Button label="Calculate" @click="calculateHolidayPay" />
    <div v-if="holidayPay !== null">
      <h3>Holiday Pay: {{ formatCurrency(holidayPay) }}</h3>
      <p>Computation:</p>
      <pre>{{ computationExplanation }}</pre>
    </div>
    <div v-if="selectedHoliday">
      <p>Date: {{ selectedHoliday.date }}</p>
      <p>Type: {{ getHolidayTypeName(selectedHoliday.type) }}</p>
    </div>
  </div>

  <h2>2024 Philippine Holidays</h2>
  <DataTable :value="holidaysWithDayOfWeek" responsiveLayout="scroll">
    <Column field="name" header="Holiday Name"></Column>
    <Column field="monthDay" header="Date"></Column>
    <!-- <Column field="dayOfWeek" header="Day of Week"></Column> -->
    <Column field="type" header="Type">
      <template #body="slotProps">
        {{ getHolidayTypeName(slotProps.data.type) }}
      </template>
    </Column>
  </DataTable>
</template>

<script setup>
import { ref, computed } from "vue";
import { HolidayTypes } from "~/utils/holidayTypes";
import holidays from "~/static/holidays.json";

const monthlyPay = ref(null);
const dailyPay = ref(null);
const hourlyPay = ref(null);
const isRestDay = ref(false);
const selectedHoliday = ref(null);
const hoursWorked = ref(null);
const holidayPay = ref(null);
const dayjs = useDayjs();
const currentDay = dayjs().format("MMMM D, YYYY");

const computationExplanation = ref("");

async function fetchHolidays() {
  holidays.value = holidays;
}

function calculateHolidayPay() {
  if (selectedHoliday.value.type === "REGULAR") {
    if (hoursWorked.value === 0) {
      // Employees who did not work on a regular holiday
      holidayPay.value = dailyPay.value || monthlyPay.value / 22;
      computationExplanation.value = `Holiday Pay = Daily Pay (or Monthly Pay / 22)
Holiday Pay = ${formatCurrency(dailyPay.value || monthlyPay.value / 22)}`;
    } else if (hoursWorked.value <= 8) {
      // Employees who worked for 8 hours or less on a regular holiday
      holidayPay.value = (dailyPay.value || monthlyPay.value / 22) * 2;
      computationExplanation.value = `Holiday Pay = (Daily Pay or Monthly Pay / 22) × 2
Holiday Pay = ${formatCurrency(dailyPay.value || monthlyPay.value / 22)} × 2
Holiday Pay = ${formatCurrency(holidayPay.value)}`;
    } else {
      // Employees who worked overtime on a regular holiday
      const overtimeHours = hoursWorked.value - 8;
      const overtimePay = hourlyPay.value * 2 * 1.3 * overtimeHours;
      holidayPay.value =
        (dailyPay.value || monthlyPay.value / 22) * 2 + overtimePay;
      computationExplanation.value = `Holiday Pay = (Daily Pay or Monthly Pay / 22) × 2 + (Hourly Pay × 2 × 1.3 × Overtime Hours)
Regular Pay = ${formatCurrency(
        dailyPay.value || monthlyPay.value / 22
      )} × 2 = ${formatCurrency((dailyPay.value || monthlyPay.value / 22) * 2)}
Overtime Pay = ${formatCurrency(
        hourlyPay.value
      )} × 2 × 1.3 × ${overtimeHours} hours = ${formatCurrency(overtimePay)}
Holiday Pay = Regular Pay + Overtime Pay
Holiday Pay = ${formatCurrency(
        (dailyPay.value || monthlyPay.value / 22) * 2
      )} + ${formatCurrency(overtimePay)}
Holiday Pay = ${formatCurrency(holidayPay.value)}`;
    }

    if (isRestDay.value) {
      if (hoursWorked.value <= 8) {
        // Employees who worked for 8 hours or less on a regular holiday falling on their rest day
        holidayPay.value = holidayPay.value + holidayPay.value * 0.3;
        computationExplanation.value += `\n\nSince the regular holiday falls on a rest day:
Additional Pay = Holiday Pay × 0.3
Additional Pay = ${formatCurrency(holidayPay.value)} × 0.3 = ${formatCurrency(
          holidayPay.value * 0.3
        )}
Total Holiday Pay = Holiday Pay + Additional Pay
Total Holiday Pay = ${formatCurrency(holidayPay.value)} + ${formatCurrency(
          holidayPay.value * 0.3
        )}
Total Holiday Pay = ${formatCurrency(holidayPay.value)}`;
      } else {
        // Employees who worked overtime on a regular holiday falling on their rest day
        const overtimeHours = hoursWorked.value - 8;
        const overtimePay = hourlyPay.value * 2 * 1.3 * 1.3 * overtimeHours;
        holidayPay.value = holidayPay.value + overtimePay;
        computationExplanation.value += `\n\nSince the regular holiday falls on a rest day and there is overtime:
Overtime Pay = Hourly Pay × 2 × 1.3 × 1.3 × Overtime Hours
Overtime Pay = ${formatCurrency(
          hourlyPay.value
        )} × 2 × 1.3 × 1.3 × ${overtimeHours} hours = ${formatCurrency(
          overtimePay
        )}
Total Holiday Pay = Holiday Pay + Overtime Pay
Total Holiday Pay = ${formatCurrency(holidayPay.value)} + ${formatCurrency(
          overtimePay
        )}
Total Holiday Pay = ${formatCurrency(holidayPay.value)}`;
      }
    }
  } else if (selectedHoliday.value.type === "SPECIAL_NON_WORKING") {
    if (hoursWorked.value === 0) {
      // Employees who did not work on a special non-working day
      holidayPay.value = 0;
      computationExplanation.value = "Holiday Pay = 0 (No work, no pay)";
    } else if (hoursWorked.value <= 8) {
      // Employees who worked for 8 hours or less on a special non-working day
      holidayPay.value = (dailyPay.value || monthlyPay.value / 22) * 1.3;
      computationExplanation.value = `Holiday Pay = (Daily Pay or Monthly Pay / 22) × 1.3
Holiday Pay = ${formatCurrency(dailyPay.value || monthlyPay.value / 22)} × 1.3
Holiday Pay = ${formatCurrency(holidayPay.value)}`;
    } else {
      // Employees who worked overtime on a special non-working day
      const overtimeHours = hoursWorked.value - 8;
      const overtimePay = hourlyPay.value * 1.3 * 1.3 * overtimeHours;
      holidayPay.value =
        (dailyPay.value || monthlyPay.value / 22) * 1.3 + overtimePay;
      computationExplanation.value = `Holiday Pay = (Daily Pay or Monthly Pay / 22) × 1.3 + (Hourly Pay × 1.3 × 1.3 × Overtime Hours)
Regular Pay = ${formatCurrency(
        dailyPay.value || monthlyPay.value / 22
      )} × 1.3 = ${formatCurrency(
        (dailyPay.value || monthlyPay.value / 22) * 1.3
      )}
Overtime Pay = ${formatCurrency(
        hourlyPay.value
      )} × 1.3 × 1.3 × ${overtimeHours} hours = ${formatCurrency(overtimePay)}
Holiday Pay = Regular Pay + Overtime Pay
Holiday Pay = ${formatCurrency(
        (dailyPay.value || monthlyPay.value / 22) * 1.3
      )} + ${formatCurrency(overtimePay)}
Holiday Pay = ${formatCurrency(holidayPay.value)}`;
    }

    if (isRestDay.value) {
      if (hoursWorked.value <= 8) {
        // Employees who worked for 8 hours or less on a special non-working day falling on their rest day
        holidayPay.value = (dailyPay.value || monthlyPay.value / 22) * 1.5;
        computationExplanation.value = `Holiday Pay = (Daily Pay or Monthly Pay / 22) × 1.5
Holiday Pay = ${formatCurrency(dailyPay.value || monthlyPay.value / 22)} × 1.5
Holiday Pay = ${formatCurrency(holidayPay.value)}`;
      } else {
        // Employees who worked overtime on a special non-working day falling on their rest day
        const overtimeHours = hoursWorked.value - 8;
        const overtimePay = hourlyPay.value * 1.5 * 1.3 * overtimeHours;
        holidayPay.value =
          (dailyPay.value || monthlyPay.value / 22) * 1.5 + overtimePay;
        computationExplanation.value = `Holiday Pay = (Daily Pay or Monthly Pay / 22) × 1.5 + (Hourly Pay × 1.5 × 1.3 × Overtime Hours)
Regular Pay = ${formatCurrency(
          dailyPay.value || monthlyPay.value / 22
        )} × 1.5 = ${formatCurrency(
          (dailyPay.value || monthlyPay.value / 22) * 1.5
        )}
Overtime Pay = ${formatCurrency(
          hourlyPay.value
        )} × 1.5 × 1.3 × ${overtimeHours} hours = ${formatCurrency(overtimePay)}
Holiday Pay = Regular Pay + Overtime Pay
Holiday Pay = ${formatCurrency(
          (dailyPay.value || monthlyPay.value / 22) * 1.5
        )} + ${formatCurrency(overtimePay)}
Holiday Pay = ${formatCurrency(holidayPay.value)}`;
      }
    }
  }
}

function formatCurrency(value) {
  return value.toLocaleString("en-PH", { style: "currency", currency: "PHP" });
}

function getHolidayTypeName(type) {
  return HolidayTypes[type];
}

const currentYear = dayjs().year();

const holidaysWithDayOfWeek = computed(() => {
  return holidays.value.map((holiday) => {
    const [month, day] = holiday.date.split("/");
    const holidayDate = dayjs(`${currentYear}-${month}-${day}`);
    return {
      ...holiday,
      monthDay: holidayDate.format("MMMM D, dddd"),
      dayOfWeek: holidayDate.format("dddd"),
    };
  });
});

fetchHolidays();
</script>
