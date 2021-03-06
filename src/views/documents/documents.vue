<template>
<v-container>

    <v-toolbar color="white" flat class="elevation-1">
        <v-toolbar-title>Documents</v-toolbar-title>
        <v-spacer></v-spacer>
        <v-btn to="/documents/add" color="primary">new document</v-btn>
    </v-toolbar>

    <v-divider class="mx-2" inset></v-divider>

    <v-data-table :loading="loading" :headers="headers" :total-items="totalDocuments" :pagination.sync="pagination" :items="documents" class="elevation-5">
        <template slot="items" slot-scope="props">
            <td><router-link :to="`documents/${props.item.id}`">{{ props.item.title }}</router-link></td>
            <td>{{ props.item.draft }}</td>
            <td>{{ props.item.createdAt }}</td>
            <td>{{ props.item.updatedAt }}</td>
            <td>
                <v-icon @click="_deleteDocument(props.item.id)" small class="mr-4">delete</v-icon>
            </td>
        </template>
    </v-data-table>

    <v-snackbar v-model="snackbar">
        Are you sure you want to remove this document?
        <v-btn flat color="primary" @click.native="deleteDocument(); snackbar = false">Yes</v-btn>
        <v-btn flat color="primary" @click.native="snackbar = false">Close</v-btn>
    </v-snackbar>

</v-container>
</template>

<script>
export default {
    name: "documents",
    data() {
        return {
            loading: "primary",
            documents: [],
            snackbar: false,
            pagination: {},
            documentId: null,
            totalDocuments: 0,
            headers: [

                {
                    text: "Title",
                    sortable: true,
                    value: "title"
                },

                {
                    text: "Draft",
                    sortable: false,
                    value: "draft"
                },

                {
                    text: "Created At",
                    sortable: true,
                    value: "createdAt"
                },

                {
                    text: "Updated At",
                    sortable: true,
                    value: "updatedAt"
                },

                {
                    text: "Actions",
                    sortable: false
                }

            ]
        }
    },

    methods: {

        _deleteDocument(id) {

            this.documentId = id;

            this.snackbar = true;

        },

        async load() {

            this.loading = true;

            let filter = {
                skip: ((this.pagination.page - 1) * this.pagination.rowsPerPage) || undefined,
                limit: this.pagination.rowsPerPage
            };

            try {

                let data = await this.$store.dispatch("getObjects", {
                    path: `documents?filter=${JSON.stringify(filter)}`
                });

                this.documents = data.rows;

                this.totalDocuments = data.count;

                this.loading = false;

            } catch (error) {

                console.error(error);

            }

        },

        async deleteDocument() {

            try {

                this.loading = true;

                await this.$store.dispatch("deleteObject", {
                    path: `documents/${this.documentId}`
                });

                this.load();

            } catch (error) {

                console.error(error);

            }

        },

    },

    async mounted() {

        this.load();

    },

    watch: {
        pagination: {
            deep: true,
            handler() {
                this.load();
            }
        }
    }

};
</script>
