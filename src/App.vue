<template>
    <v-app>
        <v-app-bar
                app
                color="primary"
                dark
                dense
        >
            <v-toolbar-title>FOXRAM</v-toolbar-title>

            <v-menu
                    left
                    bottom
            >
                <template v-slot:activator="{ on }">
                    <v-btn color="info" v-on="on" class="ml-auto mr-auto">
                        <v-icon>mdi-dots-vertical</v-icon>
                        Модули
                    </v-btn>
                </template>

                <v-list>
                    <v-list-item
                            v-for="(module,index) in modules"
                            :key="index"
                            @click="changeModule(index)"
                    >
                        <v-list-item-title>{{ module.title }}</v-list-item-title>
                    </v-list-item>
                </v-list>
            </v-menu>
        </v-app-bar>

        <v-content>
            <v-container class="fill-height" fluid >
                <v-row align="center" justify="center" no-gutters>
                    <v-col class="text-center" cols="12" sm="10" md="8">
                        <v-card raised>
                            <v-card-title>
                                {{ modules[currentModule].title }}
                            </v-card-title>

                            <v-card-text class="text-left" v-html="instruction">
                            </v-card-text>

                            <v-card-subtitle class="text-left">
                                {{ modules[currentModule].description }}
                            </v-card-subtitle>

                            <v-card-text>
                                <v-form v-model="valid">
                                    <v-container>
                                        <v-text-field
                                                v-model="input"
                                                :rules="rules"
                                                label="Formula"
                                                required
                                        />
                                        <v-btn text color="deep-purple accent-4" @click="getAnswer" :disabled="!valid">
                                            Enter
                                        </v-btn>
                                    </v-container>
                                </v-form>
                                <v-divider class="mx-4"/>
                                <v-alert color="primary" dense class="white--text">
                                    Result is: <b>{{ answer }}</b>
                                </v-alert>
                            </v-card-text>
                        </v-card>
                    </v-col>
                </v-row>
            </v-container>
        </v-content>
        <v-footer
                color="primary"
                app
        >
            <v-row justify="center" align="center">
                <v-col class="white--text text-center">МЛИТА 2020</v-col>
            </v-row>
        </v-footer>
    </v-app>
</template>

<script>

    export default {
        name: 'App',

        components: {},

        data: () => ({
            modules: [
                {title: 'title1', description: 'description1'},
                {title: 'title2', description: 'description2'},
                {title: 'title3', description: 'description3'},
                {title: 'title4', description: 'description4'},
            ],
            currentModule: 0,
            input: null,
            rules: [
                value => !!value || 'Required.',
            ],
            valid: false,
            answer: null,
            instruction: "<b>+</b> - Конъюнкция; <b>*</b> - Дизъюнкция; <b>-></b> - Импликация; <b><=></b> - Эквивалентность; <b>^</b> - XOR"
        }),
        computed: {},
        methods: {
            changeModule(index) {
                this.currentModule = index;
            },
            getAnswer() {
                 this.answer = Math.random() >= 0.5;
            }
        },
    };
</script>
