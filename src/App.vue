<template>
    <main class="age-calculator">
        <div class="container">
            <section class="age-calculator__input-values">
                <div class="age-calculator__input-group">
                    <InputField
                        type="text"
                        id="day"
                        name="day"
                        v-model="day"
                        :valid="dayValid.valid"
                        :errorMessage="dayValid.message"
                        label="Day"
                        placeholder="DD"
                        :maxLength="2"
                        :shouldFocus="focusDay"
                    />
                </div>
                <div class="age-calculator__input-group">
                    <InputField
                        type="text"
                        id="month"
                        name="month"
                        v-model="month"
                        :valid="monthValid.valid"
                        :errorMessage="monthValid.message"
                        label="Month"
                        placeholder="MM"
                        :maxLength="2"
                    />
                </div>
                <div class="age-calculator__input-group">
                    <InputField
                        type="text"
                        id="year"
                        name="year"
                        v-model="year"
                        :valid="yearValid.valid"
                        :errorMessage="yearValid.message"
                        label="Year"
                        placeholder="YYYY"
                        :maxLength="4"
                    />
                </div>
            </section>

            <section class="age-calculator-action">
                <button class="button" @click="validateInputFields" aria-label="Calculate Age">
                    <IconArrow />
                </button>
            </section>

            <section class="calculated-age">
                <div class="years">
                    <transition name="count-up">
                        <div class="data-value" key="years">
                            <span class="years__value" v-if="years !== null && dayValid.valid && monthValid.valid && yearValid.valid">
                                {{ years < 10 && years > 0 ? "0" + years : years }}
                            </span>
                            <span v-else> -- </span>
                        </div>
                    </transition>
                    <div class="data-label">years</div>
                </div>
                <div class="months">
                    <transition name="count-up">
                        <div class="data-value" key="months">
                            <span class="months__value" v-if="months !== null && dayValid.valid && monthValid.valid && yearValid.valid">
                                {{ months < 10 && months > 0 ? "0" + months : months }}
                            </span>
                            <span v-else> -- </span>
                        </div>
                    </transition>
                    <div class="data-label">months</div>
                </div>
                <div class="days">
                    <transition name="count-up">
                        <div class="data-value" key="days">
                            <span class="days__value" v-if="days !== null && dayValid.valid && monthValid.valid && yearValid.valid">
                                {{ days < 10 && days > 0 ? "0" + days : days }}
                            </span>
                            <span v-else> -- </span>
                        </div>
                    </transition>
                    <div class="data-label">days</div>
                </div>
            </section>
        </div>
    </main>
</template>

<script setup>
import {ref, onMounted} from "vue";
import InputField from "./components/InputField.vue";
import IconArrow from "./components/icons/icon-arrow.vue";

const today = new Date();

const valid = ref(false);
const loading = ref(false);

const years = ref(null);
const months = ref(null);
const days = ref(null);

const calculated = ref(false);

const focusDay = ref(false);

const day = ref("");
const month = ref("");
const year = ref("");

const dayValid = ref({message: "This field is required", valid: true});
const monthValid = ref({message: "This field is required", valid: true});
const yearValid = ref({message: "This field is required", valid: true});

const validateField = async (field, error, validator) => {
    if (field.value === "") {
        valid.value = false;
        error.value.valid = false;
        error.value.message = "This field is required";
        return;
    }

    const validationResult = await validator();

    if (validationResult.valid) {
        valid.value = true;
        error.value.valid = true;
    } else {
        valid.value = false;
        error.value.valid = false;
        error.value.message = validationResult.message;
    }
};

const validateDay = async () => {
    if (day.value < 1 || day.value > 31) {
        return {valid: false, message: "Must be a valid day"};
    }

    const daysInMonth = new Date(year.value, month.value, 0).getDate();

    if (day.value > daysInMonth) {
        return {valid: false, message: "Must be a valid day"};
    }

    return {valid: true};
};

const validateMonth = async () => {
    if (month.value < 1 || month.value > 12) {
        return {valid: false, message: "Must be a valid month"};
    }

    return {valid: true};
};

const validateYear = async () => {
    if (year.value > today.getFullYear()) {
        return {valid: false, message: "Must be a valid year"};
    } else if (year.value < 1900) {
        return {valid: false, message: "Year must be after 1900"};
    }

    return {valid: true};
};

const validateInputFields = async () => {
    await validateField(day, dayValid, validateDay);
    await validateField(month, monthValid, validateMonth);
    await validateField(year, yearValid, validateYear);

    if (valid.value) {
        calculateAge();
    }
};

const resetCalculator = () => {
    days.value = null;
    months.value = null;
    years.value = null;
};

const countUp = (targetValue, property) => {
    const startValue = 0;
    const steps = 60;
    const stepValue = (targetValue - startValue) / steps;
    let currentValue = startValue;

    const update = () => {
        if (currentValue <= targetValue) {
            property.value = Math.floor(currentValue);
            currentValue += stepValue;
            requestAnimationFrame(update);
        }
    };

    update();
};

const calculateAge = () => {
    loading.value = true;

    resetCalculator();

    if (!valid.value) {
        return;
    }

    const birthDate = new Date(`${month.value}-${day.value}-${year.value}`);

    countUp(today.getFullYear() - birthDate.getFullYear(), years);
    countUp(today.getMonth() - birthDate.getMonth(), months);

    let daysDiff = today.getDate() - birthDate.getDate();

    if (daysDiff < 0) {
        months.value--;
        daysDiff += new Date(today.getFullYear(), today.getMonth(), 0).getDate();
    }

    countUp(daysDiff, days);

    calculated.value = true;
};

onMounted(() => {
    focusDay.value = true;
});
</script>
