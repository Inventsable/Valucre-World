<template>
    <v-dialog
        v-model="state"
        width="500"
        >
        <v-card>
            <v-card-text>
                <v-text-field
                    label="Link to current view"
                    append-outer-icon="assignment_turned_in"
                    v-model="result"
                ></v-text-field>
            </v-card-text>
            <v-divider></v-divider>
        </v-card>
    </v-dialog>
</template>

<script>
export default {
    name: 'linkdialog',
    data: () => ({
        state: false,
        homepage: 'https://valucre.world/',
        // homepage: 'http://localhost:8080/',
        route: 'z4toz6'
    }),
    computed: {
        result() {
            return `${this.homepage}${this.route}`;
        }
    },
    mounted() {
        console.log('Hello world from link dialog')
    }
}
</script>

