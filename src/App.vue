<template>
    <v-app>
        <v-app-bar
                app
                color="primary"
                dark
                dense
        >
            <v-toolbar-title class="mx-auto">FOXRAM</v-toolbar-title>
        </v-app-bar>

        <v-content>
            <v-container class="fill-height" fluid>
                <v-row align="center" justify="center" no-gutters>
                    <v-col class="text-center" cols="12" sm="10" md="8">
                        <v-card raised>
                            <v-tabs
                                    v-model="tab"
                                    background-color="transparent"
                                    color="basil"
                                    grow
                            >
                                <v-tab
                                        v-for="(module, index) in modules"
                                        :key="index"
                                >
                                    {{ module.title }}
                                </v-tab>
                            </v-tabs>

                            <v-card-title>
                                {{ modules[tab].title }}
                            </v-card-title>

                            <v-card-text class="text-left" v-html="instruction"/>

                            <v-card-subtitle class="text-left">
                                {{ modules[tab].description }}
                            </v-card-subtitle>

                            <v-card-text>
                                <v-tabs-items v-model="tab">
                                    <v-tab-item v-for="(module, index) in modules" :key="index">
                                        <module0 v-if="index===0"/>
                                        <module1 v-if="index===1"/>
                                        <module2 v-if="index===2"/>
                                        <module3 v-if="index===3"/>
                                    </v-tab-item>
                                </v-tabs-items>
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
    import module0 from "./components/module0";
    import module1 from "./components/module1";
    import module2 from "./components/module2";
    import module3 from "./components/module3";

    export default {
        name: 'App',

        components: {
            module0,
            module1,
            module2,
            module3
        },

        data: () => ({
            modules: [
                {title: 'Общезначимость', description: 'Модуль для проверки функции на общезначимость (то есть при любых значениях аргументов результат - истина)'},
                {title: 'Противоречивость', description: 'Модуль для проверки функции на противоречивость (то есть при любых значениях аргументов результат - ложь)'},
                {title: 'Основная теорема логического вывода', description: 'Использование формулы F1∧F2∧..∧Fn→G'},
                {title: 'Метод резолюций', description: 'Использование метода резолюции для проверки истинности высказывания'},
            ],
            operations: [
                {operation: 'Отрицание', sign: '!'},
                {operation: 'Конъюнкция', sign: '∧'},
                {operation: 'Дизъюнкция', sign: '∨'},
                {operation: 'Импликация', sign: '→'},
                {operation: 'Эквиваленция', sign: '⇔'},
                {operation: 'XOR', sign: '⊕'},
            ],
            tab: 0,
            currentModule: 0,
            instruction: "<b>!</b> - Отрицание; <b>∧</b> - Конъюнкция; <b>∨</b> - Дизъюнкция; <b>→</b> - Импликация; <b>⇔</b> - Эквивалентность; <b>⊕</b> - XOR"
        }),
        computed: {},
        methods: {

        },
    };
</script>
