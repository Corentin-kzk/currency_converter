<script setup>
import { reactive, ref, watch } from "vue";
import { useQueryClient, useMutation, useQuery } from "@tanstack/vue-query";
import { QUERY_KEY_PAIRS, getPair, updatePair, createPair } from "../services/api/pairs"
import { getCurrencies, QUERY_KEY_CURRENCY } from '../services/api/currencies'
import router from "@/router";

const queryClient = useQueryClient()

const form = reactive({
    from: "",
    to: "",
    conversion_rate: ""
})
const errors = ref(null)

const mutation = useMutation({
    mutationFn: createPair,
    onSuccess: () => {
        queryClient.invalidateQueries({ queryKey: [QUERY_KEY_PAIRS] })
        router.push('/admin')
    },
    onError: (err) => {
        errors.value = err?.message
    }
})

const currenciesSelectValue = ref([])


const { isLoading, isSuccess } = useQuery({
    queryKey: [QUERY_KEY_PAIRS],
    queryFn: () => getCurrencies(),
    onSuccess: (data) => {
        currenciesSelectValue.value = data.map((currency) => currency.name)
    }
})

function onUpdateSubmit() {
    mutation.mutateAsync({ from: form.from, to: form.to, conversion_rate: form.conversion_rate })
}

watch(form, () => {
    if (form.from && form.from === form.to) {
        errors.value =  "The currencies cannot be the same." 
        return 
    }
    errors.value = null
});


const rules = {
    required: (value) => value?.trim() !== "" ? true : "Required",
    mustBeNumber: (value) => !isNaN(value) ? true : "the value must be a number"
}
</script>

<template>
    <v-container>
        <v-row>
            <h1>Create Pair</h1>
        </v-row>
        <v-row v-if="isLoading && !isSuccess" align="center">
            <v-progress-circular :size="50" color="primary" indeterminate></v-progress-circular>
        </v-row>
        <v-form @submit.prevent="onUpdateSubmit" v-else>
            <v-row align="center">
                <v-col>
                    <v-responsive class="mx-auto my-4">

                        <v-select :items="currenciesSelectValue" v-model="form.from" clearable hide-details="auto"
                            label="from" :rules="[rules.required]"></v-select>
                    </v-responsive>
                </v-col>
                <v-col>
                    <v-responsive class="mx-auto my-4">
                        <v-select :items="currenciesSelectValue" v-model="form.to" clearable hide-details="auto" label="to"
                            :rules="[rules.required]"></v-select>
                    </v-responsive>
                </v-col>
            </v-row>
            <v-alert type="error" v-if="!!errors" :text="errors" class="mb-2" />

            <v-responsive class="mx-auto my-4">
                <v-text-field v-model="form.conversion_rate" clearable hide-details="auto" label="Conversion rate"
                    :rules="[rules.required, rules.mustBeNumber]"></v-text-field>
            </v-responsive>
            <v-btn block type="submit" :disabled="!!errors" class="mb-8" color="blue" size="large"
                variant="green">
                Add new pair
            </v-btn>
        </v-form>
    </v-container></template>