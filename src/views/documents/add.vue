<template>
<v-card>

    <v-card-title primary-title>
        <h1>New Document</h1>
    </v-card-title>

    <v-card-text>

        <v-form @submit.prevent="save">

            <v-select v-model="document.language" :items="['english', 'french']" label="Language" :error-messages="errors.language">
            </v-select>

            <v-select v-model="document.parentDocumentId" :items="documents" item-text="title" item-value="id" label="Parent Document">
            </v-select>

            <v-switch label="Draft" :error-messages="errors.draft" v-model="document.draft" color="primary"></v-switch>

            <v-text-field type="text" :error-messages="errors.title" v-model="document.title" label="Title"></v-text-field>

            <quill-editor class="mt-3" v-model="document.content" ref="documentEditor" :options="editorOptions"></quill-editor>

            <v-btn color="primary" type="submit" block class="mt-3" :disabled="disabled" :loading="loading">save</v-btn>

        </v-form>

    </v-card-text>

</v-card>
</template>

<script>
import "quill/dist/quill.core.css";
import "quill/dist/quill.snow.css";

import {
    quillEditor
} from "vue-quill-editor";

export default {
    name: "add-document",
    components: {
        quillEditor
    },
    data() {
        return {
            loading: false,
            disabled: true,
            document: {
                draft: true,
                title: undefined,
                content: undefined,
                language: undefined,
                parentDocumentId: undefined
            },
            documents: [],
            errors: {},
            editorOptions: {}
        }
    },
    methods: {
        async save() {

            this.loading = true;

            try {

                this.document = await this.$store.dispatch("saveObject", {
                    path: "documents",
                    data: this.document
                });

                this.loading = false;

                this.$router.replace(`/documents/${this.document.id}`);

            } catch (error) {

                this.loading = false;

                console.error(error);

            }

        }
    },

    async created() {

        try {

            let documents = await this.$store.dispatch("getObjects", {
                path: "/documents"
            });

            this.documents = documents.rows;

        } catch (error) {

            console.error(error);

        }

    },

    watch: {
        document: {
            deep: true,
            handler() {
                let constraints = {

                    language: {
                        presence: true,
                        inclusion: ["english", "french"]
                    },

                    draft: {
                        presence: true,
                        inclusion: [true, false]
                    },

                    title: {
                        presence: true,
                        length: {
                            minimum: 2,
                            maximum: 100,
                            message: "Title must be 2 to 100 characters"
                        }
                    },

                    content: {
                        presence: true,
                        length: {
                            minimum: 2,
                            message: "Content must be more than 2 characters"
                        }
                    }

                };

                let errors = this.$validate(this.document, constraints);

                this.errors = errors || {};

                this.disabled = errors !== undefined;
            }
        }
    }
}
</script>
