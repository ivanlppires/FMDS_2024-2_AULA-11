<template>
    <v-container>
        <v-data-table-server v-model:items-per-page="itemsPerPage" :headers="headers" :items="serverItems"
            :items-length="totalItems" :loading="loading" :search="search" item-value="name"
            @update:options="loadItems"
            ></v-data-table-server>
    </v-container>
</template>
<script setup>
import { ref } from 'vue'
const itemsPerPage = ref(10)
const headers = ref([
    { title: 'Nome', key: 'name' },
    { title: 'E-mail', key: 'email' }
])
const serverItems = ref([])
const totalItems = ref(0)
const loading = ref(false)
const search = ref('')
const loadItems = async (options) => {
    loading.value = true;
    let url = 'http://localhost:5000/client?1';
    if (options) {
        // SORT BY
        console.log(options.sortBy)
        if (options.sortBy.length > 0) {
            url += '&sortKey=' + options.sortBy[0].key
            url += '&sortOrder=' + options.sortBy[0].order
        }
        // LIMIT (PAGE)
        url += '&page=' + options.page || 1
        url += '&limit=' + options.itemsPerPage || itemsPerPage.value
    }
    const response = await fetch(url);
    const result = await response.json();
    serverItems.value = result.data;
    totalItems.value = result.total;
    loading.value = false;

}
</script>