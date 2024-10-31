<template>
    <v-container>
        <v-data-table-server v-model:items-per-page="itemsPerPage" :headers="headers" :items="serverItems"
            :items-length="totalItems" :loading="loading" :search="search" item-value="name"
            @update:options="loadItems">
            <!-- search field in v-slot -->
            <template v-slot:top>
                <v-toolbar flat>
                    <v-toolbar-title>
                        Controle de Cliente
                    </v-toolbar-title>
                    <v-toolbar-items>
                        <v-btn @click="addItem">Adicionar</v-btn>
                    </v-toolbar-items>
                </v-toolbar>
                <v-container>
                    <v-text-field v-model="searchName" label="Pesquisar...  " variant="underlined"></v-text-field>
                </v-container>
            </template>
            <template v-slot:item.action="{ item }">
                <v-btn icon="mdi-pencil" @click="editItem(item)" class="mr-2" />
                <v-btn icon="mdi-delete" @click="remove(item.uuid)" />
            </template>
        </v-data-table-server>
    </v-container>
    <v-dialog v-model="dialog" max-width="600">
        <v-card>
            <v-card-title>Cliente</v-card-title>
            <v-card-text>
                <v-text-field label="Nome" v-model="client.name" />
                <v-text-field label="E-mail" v-model="client.email" />
            </v-card-text>
            <v-card-actions>
                <v-btn color="error" @click="dialog = false">Cancelar</v-btn>
                <v-spacer />
                <v-btn color="primary" @click="save">Salvar</v-btn>
            </v-card-actions>
        </v-card>
    </v-dialog>
</template>
<script setup>
import { ref, watch } from 'vue'
import debounce from 'lodash/debounce';
const client = ref({});
const itemsPerPage = ref(10)
const page = ref(1)
const sortBy = ref([])
const dialog = ref(false)
const headers = ref([
    { title: 'Nome', key: 'name' },
    { title: 'E-mail', key: 'email' },
    { title: 'Ações', key: 'action' }
])
const serverItems = ref([])
const totalItems = ref(0)
const loading = ref(false)
const search = ref('')
const searchName = ref('')
const loadItems = async (options) => {
    loading.value = true;
    let url = 'http://localhost:5000/client?1';
    if (options) {
        // SORT BY
        page.value = options.page;
        sortBy.value = options.sortBy;
        if (options.sortBy.length > 0) {
            url += '&sortKey=' + options.sortBy[0].key
            url += '&sortOrder=' + options.sortBy[0].order
        }
        if (options.search.length > 0) {
            url += '&search=' + options.search
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
watch(searchName, debounce(() => {
    search.value = searchName.value
}, 500))
const remove = async (uuid) => {
    await fetch('http://localhost:5000/client/' + uuid, {
        method: 'DELETE'
    })
    loadItems({ page: page.value, itemsPerPage: itemsPerPage.value, sortBy: sortBy.value, search: searchName.value })
}
const save = () => {
    fetch(
        'http://localhost:5000/client',
        {
            method: (client.value.uuid) ? 'PUT' : 'POST',
            headers: {
                'Content-Type': 'application/json',
            },
            body: JSON.stringify(client.value)
        }
    ).then().then(
        () => {
            client.value = {}
            dialog.value = false;
            loadItems({ page: page.value, itemsPerPage: itemsPerPage.value, sortBy: sortBy.value, search: searchName.value })
        }
    ).catch(
        error => {
            console.log(error)
        }
    );

}
const editItem = (item) => {
    client.value = { ...item }
    dialog.value = true;
}
const addItem = () => {
    client.value = {}
    dialog.value = true;
}
</script>